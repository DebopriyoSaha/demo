import { useEffect, useMemo, useState } from "react";
import { useNavigate } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { getCurrentUser } from "@/lib/auth";
import ConfirmDialog from "@/components/ConfirmDialog";

type Task = {
  id: string;
  title: string;
  due?: string; // ISO date (YYYY-MM-DD) or undefined
  completed: boolean;
  createdAt: string; // ISO
  profile: "work" | "personal";
};

const PROFILE: Task["profile"] = "work"; // this page is for Work

// ----- storage helpers (per-user bucket) -----
function currentUserId(): string | null {
  const u = getCurrentUser();
  return u ? u.id : null;
}
function key(uid: string) {
  return `journal.todos.${uid}`;
}
function loadAll(): Task[] {
  const uid = currentUserId();
  if (!uid) return [];
  try {
    const raw = localStorage.getItem(key(uid));
    return raw ? (JSON.parse(raw) as Task[]) : [];
  } catch {
    return [];
  }
}
function saveAll(tasks: Task[]) {
  const uid = currentUserId();
  if (!uid) return;
  localStorage.setItem(key(uid), JSON.stringify(tasks));
}

// ----- Add Task inline form -----
function AddTaskInline({
  onAdd,
  onCancel,
}: {
  onAdd: (t: { title: string; due?: string }) => void;
  onCancel: () => void;
}) {
  const [title, setTitle] = useState("");
  const [due, setDue] = useState<string>("");
  const [err, setErr] = useState<string | null>(null);

  function submit(e: React.FormEvent) {
    e.preventDefault();
    if (!title.trim()) {
      setErr("Task title is required.");
      return;
    }
    onAdd({ title: title.trim(), due: due || undefined });
  }

  return (
    <form
      onSubmit={submit}
      className="w-full max-w-xl rounded-xl border border-neutral-200 bg-white/80 backdrop-blur-sm p-4 shadow-sm"
    >
      {err && (
        <div className="mb-3 rounded-md border border-red-200 bg-red-50 px-3 py-2 text-sm text-red-700">
          {err}
        </div>
      )}
      <div className="grid gap-3 md:grid-cols-3">
        <div className="md:col-span-2">
          <label className="block text-sm font-medium text-neutral-800">Task</label>
          <input
            className="mt-1 w-full rounded-md border border-neutral-300 bg-white/90 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            value={title}
            onChange={(e) => {
              setTitle(e.target.value);
              setErr(null);
            }}
            placeholder="e.g., Prepare deck for client call"
            autoFocus
          />
        </div>
        <div>
          <label className="block text-sm font-medium text-neutral-800">Deadline</label>
          <input
            type="date"
            value={due}
            onChange={(e) => setDue(e.target.value)}
            className="mt-1 w-full rounded-md border border-neutral-300 bg-white/90 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
          />
        </div>
      </div>

      <div className="mt-3 flex items-center gap-2">
        <Button type="submit">Add</Button>
        <Button type="button" variant="ghost" onClick={onCancel}>
          Cancel
        </Button>
      </div>
    </form>
  );
}

export default function Todo() {
  const navigate = useNavigate();
  const [tasks, setTasks] = useState<Task[]>([]);
  const [showAdd, setShowAdd] = useState(false);
  const [tab, setTab] = useState<"active" | "completed">("active");
  const [deleteId, setDeleteId] = useState<string | null>(null);

  // auth + work profile context
  useEffect(() => {
    if (!currentUserId()) {
      navigate("/login", { replace: true });
      return;
    }
    // pin work profile while in this page
    localStorage.setItem("journal.selectedProfile", "work");
  }, [navigate]);

  // load
  useEffect(() => {
    setTasks(loadAll());
  }, []);

  // derived counts
  const activeCount = useMemo(() => tasks.filter((t) => !t.completed && t.profile === PROFILE).length, [tasks]);
  const completedCount = useMemo(() => tasks.filter((t) => t.completed && t.profile === PROFILE).length, [tasks]);

  // filter + sort
  const visible = useMemo(() => {
    const mine = tasks.filter((t) => t.profile === PROFILE && (tab === "active" ? !t.completed : t.completed));
    // sort by due date ASC; tasks with no due date go to bottom
    return mine.sort((a, b) => {
      if (!a.due && !b.due) return a.createdAt.localeCompare(b.createdAt);
      if (!a.due) return 1;
      if (!b.due) return -1;
      return a.due.localeCompare(b.due);
    });
  }, [tasks, tab]);

  function addTask(input: { title: string; due?: string }) {
    const now = new Date().toISOString();
    const next: Task = {
      id: crypto.randomUUID(),
      title: input.title,
      due: input.due,
      completed: false,
      createdAt: now,
      profile: PROFILE,
    };
    const updated = [next, ...tasks];
    setTasks(updated);
    saveAll(updated);
    setShowAdd(false);
  }

  function toggleTask(id: string, done: boolean) {
    const updated = tasks.map((t) => (t.id === id ? { ...t, completed: done } : t));
    setTasks(updated);
    saveAll(updated);
  }

  function removeTask(id: string) {
    const updated = tasks.filter((t) => t.id !== id);
    setTasks(updated);
    saveAll(updated);
    setDeleteId(null);
  }

  return (
    <PaperBackground>
      <div className="min-h-screen w-full px-4 md:px-8 py-6">
        {/* Header */}
        <div className="mb-4 flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
          <div>
            <h1 className="text-2xl font-semibold text-neutral-900">To-do</h1>
            <p className="text-sm text-neutral-600">Track work tasks with deadlines.</p>
          </div>
          <div className="flex items-center gap-2">
            <div className="inline-flex rounded-lg border border-neutral-200 bg-white/60 backdrop-blur-sm p-1">
              <button
                className={`px-3 py-1.5 rounded-md text-sm ${tab === "active" ? "bg-neutral-900 text-white" : "text-neutral-800 hover:bg-neutral-100"}`}
                onClick={() => setTab("active")}
              >
                Active ({activeCount})
              </button>
              <button
                className={`px-3 py-1.5 rounded-md text-sm ${tab === "completed" ? "bg-neutral-900 text-white" : "text-neutral-800 hover:bg-neutral-100"}`}
                onClick={() => setTab("completed")}
              >
                Completed ({completedCount})
              </button>
            </div>

            <Button onClick={() => setShowAdd(true)}>Add Task</Button>
          </div>
        </div>

        {/* Add form */}
        {showAdd && (
          <div className="mb-4">
            <AddTaskInline onAdd={addTask} onCancel={() => setShowAdd(false)} />
          </div>
        )}

        {/* List */}
        <div className="rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm">
          {visible.length === 0 ? (
            <div className="px-4 py-10 text-center text-neutral-500">
              {tab === "active"
                ? "No active tasks. Enjoy the calm or add one!"
                : "No completed tasks yet."}
            </div>
          ) : (
            <ul className="divide-y divide-neutral-200">
              {visible.map((t) => (
                <li key={t.id} className="px-4 py-3 flex items-center gap-3">
                  <input
                    aria-label="Mark complete"
                    type="checkbox"
                    checked={t.completed}
                    onChange={(e) => toggleTask(t.id, e.target.checked)}
                    className="h-4 w-4 rounded border-neutral-300"
                  />
                  <div className="flex-1">
                    <div className={`text-sm ${t.completed ? "line-through text-neutral-500" : "text-neutral-900"}`}>
                      {t.title}
                    </div>
                    <div className="text-xs text-neutral-600">
                      {t.due ? `Due ${t.due}` : "No deadline"}
                    </div>
                  </div>
                  <button
                    className="text-sm text-neutral-700 hover:underline underline-offset-2"
                    onClick={() => setDeleteId(t.id)}
                  >
                    Delete
                  </button>
                </li>
              ))}
            </ul>
          )}
        </div>
      </div>

      {/* Confirm delete dialog */}
      <ConfirmDialog
        open={!!deleteId}
        title="Delete this task?"
        description="This cannot be undone."
        confirmText="Delete"
        cancelText="Cancel"
        onCancel={() => setDeleteId(null)}
        onConfirm={() => {
          if (deleteId) removeTask(deleteId);
        }}
      />
    </PaperBackground>
  );
}
