import { useEffect, useMemo, useState } from "react";
import { useNavigate } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { getCurrentUser } from "@/lib/auth";
import ConfirmDialog from "@/components/ConfirmDialog";

type GrowthType = "Certification" | "Skill" | "Experience";
type GrowthItem = {
  id: string;
  title: string;
  type: GrowthType;
  start?: string;   // YYYY-MM-DD
  target?: string;  // YYYY-MM-DD
  progress: number; // 0..100
  notes?: string;
  completed: boolean;
  createdAt: string; // ISO
  profile: "work" | "personal"; // we'll focus on work here
};

const PROFILE: GrowthItem["profile"] = "work";

// ---------- storage (per user) ----------
function currentUserId(): string | null {
  const u = getCurrentUser();
  return u ? u.id : null;
}
function key(uid: string) {
  return `journal.growth.${uid}`;
}
function loadAll(): GrowthItem[] {
  const uid = currentUserId();
  if (!uid) return [];
  try {
    const raw = localStorage.getItem(key(uid));
    return raw ? (JSON.parse(raw) as GrowthItem[]) : [];
  } catch {
    return [];
  }
}
function saveAll(items: GrowthItem[]) {
  const uid = currentUserId();
  if (!uid) return;
  localStorage.setItem(key(uid), JSON.stringify(items));
}

// ---------- Add/Edit form ----------
function GrowthForm({
  initial,
  onSave,
  onCancel,
}: {
  initial?: Partial<GrowthItem>;
  onSave: (data: Omit<GrowthItem, "id" | "createdAt" | "completed" | "profile"> & { completed?: boolean }) => void;
  onCancel: () => void;
}) {
  const [title, setTitle] = useState(initial?.title ?? "");
  const [type, setType] = useState<GrowthType>((initial?.type as GrowthType) ?? "Certification");
  const [start, setStart] = useState<string>(initial?.start ?? "");
  const [target, setTarget] = useState<string>(initial?.target ?? "");
  const [progress, setProgress] = useState<number>(typeof initial?.progress === "number" ? initial!.progress : 0);
  const [notes, setNotes] = useState<string>(initial?.notes ?? "");
  const [err, setErr] = useState<string | null>(null);

  function submit(e: React.FormEvent) {
    e.preventDefault();
    if (!title.trim()) {
      setErr("Title is required.");
      return;
    }
    onSave({
      title: title.trim(),
      type,
      start: start || undefined,
      target: target || undefined,
      progress: Math.max(0, Math.min(100, progress)),
      notes: notes?.trim() ? notes.trim() : undefined,
    });
  }

  return (
    <form
      onSubmit={submit}
      className="w-full max-w-3xl rounded-xl border border-neutral-200 bg-white/80 backdrop-blur-sm p-5 shadow-sm"
    >
      {err && (
        <div className="mb-3 rounded-md border border-red-200 bg-red-50 px-3 py-2 text-sm text-red-700">
          {err}
        </div>
      )}

      <div className="grid gap-4 md:grid-cols-2">
        <div>
          <label className="block text-sm font-medium text-neutral-800">Title</label>
          <input
            className="mt-1 w-full rounded-md border border-neutral-300 bg-white/90 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            value={title}
            onChange={(e) => {
              setTitle(e.target.value);
              setErr(null);
            }}
            placeholder="e.g., AWS Solutions Architect Associate"
            autoFocus
          />
        </div>

        <div>
          <label className="block text-sm font-medium text-neutral-800">Type</label>
          <select
            className="mt-1 w-full rounded-md border border-neutral-300 bg-white/90 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            value={type}
            onChange={(e) => setType(e.target.value as GrowthType)}
          >
            <option>Certification</option>
            <option>Skill</option>
            <option>Experience</option>
          </select>
        </div>

        <div>
          <label className="block text-sm font-medium text-neutral-800">Start date</label>
          <input
            type="date"
            className="mt-1 w-full rounded-md border border-neutral-300 bg-white/90 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            value={start}
            onChange={(e) => setStart(e.target.value)}
          />
        </div>

        <div>
          <label className="block text-sm font-medium text-neutral-800">Target date</label>
          <input
            type="date"
            className="mt-1 w-full rounded-md border border-neutral-300 bg-white/90 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            value={target}
            onChange={(e) => setTarget(e.target.value)}
          />
        </div>
      </div>

      <div className="mt-3">
        <label className="block text-sm font-medium text-neutral-800">
          Progress <span className="text-neutral-500">({progress}%)</span>
        </label>
        <input
          type="range"
          min={0}
          max={100}
          value={progress}
          onChange={(e) => setProgress(parseInt(e.target.value || "0"))}
          className="mt-1 w-full"
        />
      </div>

      <div className="mt-3">
        <label className="block text-sm font-medium text-neutral-800">Notes (optional)</label>
        <textarea
          className="mt-1 w-full min-h-[120px] rounded-md border border-neutral-300 bg-white/90 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
          value={notes}
          onChange={(e) => setNotes(e.target.value)}
          placeholder="Brief plan, resources, or what you learned so far…"
        />
      </div>

      <div className="mt-4 flex items-center gap-2">
        <Button type="submit">Save</Button>
        <Button type="button" variant="ghost" onClick={onCancel}>
          Cancel
        </Button>
      </div>
    </form>
  );
}

// ---------- Card ----------
function ProgressBar({ value }: { value: number }) {
  return (
    <div className="h-2 w-full rounded-full bg-neutral-200 overflow-hidden">
      <div
        className="h-full bg-neutral-900"
        style={{ width: `${Math.max(0, Math.min(100, value))}%` }}
      />
    </div>
  );
}

export default function Growth() {
  const navigate = useNavigate();
  const [items, setItems] = useState<GrowthItem[]>([]);
  const [tab, setTab] = useState<"active" | "completed">("active");
  const [showForm, setShowForm] = useState(false);
  const [editingId, setEditingId] = useState<string | null>(null);
  const [deleteId, setDeleteId] = useState<string | null>(null);

  // auth + work profile context
  useEffect(() => {
    if (!currentUserId()) {
      navigate("/login", { replace: true });
      return;
    }
    localStorage.setItem("journal.selectedProfile", "work");
  }, [navigate]);

  // load
  useEffect(() => {
    setItems(loadAll());
  }, []);

  const activeCount = useMemo(
    () => items.filter((i) => !i.completed && i.profile === PROFILE).length,
    [items]
  );
  const completedCount = useMemo(
    () => items.filter((i) => i.completed && i.profile === PROFILE).length,
    [items]
  );

  const visible = useMemo(() => {
    const mine = items.filter(
      (i) => i.profile === PROFILE && (tab === "active" ? !i.completed : i.completed)
    );
    // sort by target date ASC; empty target → bottom
    return mine.sort((a, b) => {
      if (!a.target && !b.target) return a.createdAt.localeCompare(b.createdAt);
      if (!a.target) return 1;
      if (!b.target) return -1;
      return a.target.localeCompare(b.target);
    });
  }, [items, tab]);

  function openAdd() {
    setEditingId(null);
    setShowForm(true);
  }
  function openEdit(id: string) {
    setEditingId(id);
    setShowForm(true);
  }

  function handleSave(form: Omit<GrowthItem, "id" | "createdAt" | "completed" | "profile"> & { completed?: boolean }) {
    const now = new Date().toISOString();

    if (editingId) {
      const updated = items.map((i) =>
        i.id === editingId
          ? {
              ...i,
              ...form,
              progress: Math.max(0, Math.min(100, form.progress)),
            }
          : i
      );
      setItems(updated);
      saveAll(updated);
    } else {
      const next: GrowthItem = {
        id: crypto.randomUUID(),
        title: form.title,
        type: form.type,
        start: form.start,
        target: form.target,
        progress: Math.max(0, Math.min(100, form.progress)),
        notes: form.notes,
        completed: !!form.completed || form.progress >= 100,
        createdAt: now,
        profile: PROFILE,
      };
      const updated = [next, ...items];
      setItems(updated);
      saveAll(updated);
    }
    setShowForm(false);
    setEditingId(null);
  }

  function markComplete(id: string, done: boolean) {
    const updated = items.map((i) => (i.id === id ? { ...i, completed: done, progress: done ? 100 : i.progress } : i));
    setItems(updated);
    saveAll(updated);
  }

  function remove(id: string) {
    const updated = items.filter((i) => i.id !== id);
    setItems(updated);
    saveAll(updated);
    setDeleteId(null);
  }

  function getById(id: string | null) {
    return items.find((i) => i.id === id) || null;
  }

  return (
    <PaperBackground>
      <div className="min-h-screen w-full px-4 md:px-8 py-6">
        {/* Header */}
        <div className="mb-4 flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
          <div>
            <h1 className="text-2xl font-semibold text-neutral-900">Growth</h1>
            <p className="text-sm text-neutral-600">Track certifications, skills, and experiences.</p>
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

            <Button onClick={openAdd}>Add Goal</Button>
          </div>
        </div>

        {/* Add/Edit form */}
        {showForm && (
          <div className="mb-4">
            <GrowthForm
              initial={editingId ? getById(editingId) ?? undefined : undefined}
              onSave={handleSave}
              onCancel={() => {
                setShowForm(false);
                setEditingId(null);
              }}
            />
          </div>
        )}

        {/* Cards */}
        <div className="grid gap-4 md:grid-cols-2">
          {visible.length === 0 ? (
            <div className="col-span-full px-4 py-10 text-center text-neutral-500">
              {tab === "active"
                ? "No active growth items yet. Add your first goal!"
                : "No completed items yet."}
            </div>
          ) : (
            visible.map((g) => (
              <div
                key={g.id}
                className="rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm p-4"
              >
                <div className="flex items-start justify-between gap-3">
                  <div>
                    <div className="text-base font-semibold text-neutral-900">{g.title}</div>
                    <div className="text-xs text-neutral-600">
                      {g.type}
                      {g.start ? ` · Start ${g.start}` : ""}{g.target ? ` · Target ${g.target}` : ""}
                    </div>
                  </div>
                  <div className="text-sm text-neutral-700">{g.progress}%</div>
                </div>

                <div className="mt-2"><ProgressBar value={g.progress} /></div>

                {g.notes && <div className="mt-2 text-sm text-neutral-800 whitespace-pre-wrap">{g.notes}</div>}

                <div className="mt-3 flex items-center gap-2">
                  <Button
                    variant="outline"
                    onClick={() => openEdit(g.id)}
                  >
                    Edit
                  </Button>
                  <Button
                    variant="ghost"
                    onClick={() => markComplete(g.id, !g.completed)}
                  >
                    {g.completed ? "Mark active" : "Mark complete"}
                  </Button>
                  <Button variant="ghost" onClick={() => setDeleteId(g.id)}>
                    Delete
                  </Button>
                </div>
              </div>
            ))
          )}
        </div>
      </div>

      <ConfirmDialog
        open={!!deleteId}
        title="Delete this item?"
        description="This action cannot be undone."
        confirmText="Delete"
        cancelText="Cancel"
        onCancel={() => setDeleteId(null)}
        onConfirm={() => {
          if (deleteId) remove(deleteId);
        }}
      />
    </PaperBackground>
  );
}
