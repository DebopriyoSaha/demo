import { useEffect, useMemo, useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { logout, getCurrentUser } from "@/lib/auth";

type Category = "A" | "B" | "C";

type Entry = {
  id: string;
  name: string;
  body?: string;
  createdAt: string;    // ISO
  updatedAt: string;    // ISO
  profile: "personal" | "work";
  category?: Category;  // only for work entries
};

function currentUserId(): string | null {
  const u = getCurrentUser();
  return u ? u.id : null;
}
function userEntriesKey(uid: string) {
  return `journal.entries.${uid}`;
}
function loadAllEntries(): Entry[] {
  const uid = currentUserId();
  if (!uid) return [];
  try {
    const raw = localStorage.getItem(userEntriesKey(uid));
    return raw ? (JSON.parse(raw) as Entry[]) : [];
  } catch {
    return [];
  }
}
function saveAllEntries(entries: Entry[]) {
  const uid = currentUserId();
  if (!uid) return;
  localStorage.setItem(userEntriesKey(uid), JSON.stringify(entries));
}
function seedIfEmpty() {
  const current = loadAllEntries();
  const hasAnyWork = current.some((e) => e.profile === "work");
  if (hasAnyWork) return;

  const now = new Date();
  const iso = (d: Date) => d.toISOString();
  const daysAgo = (n: number) => {
    const d = new Date();
    d.setDate(d.getDate() - n);
    return d;
  };

  const seeded: Entry[] = [
    {
      id: crypto.randomUUID(),
      name: "Sprint planning notes",
      body: "Planned Sprint 14. Focus on feature X.",
      createdAt: iso(daysAgo(5)),
      updatedAt: iso(daysAgo(4)),
      profile: "work",
      category: "A",
    },
    {
      id: crypto.randomUUID(),
      name: "Client call recap",
      body: "Action items: follow up with legal, update timeline.",
      createdAt: iso(daysAgo(2)),
      updatedAt: iso(daysAgo(1)),
      profile: "work",
      category: "B",
    },
  ];

  saveAllEntries([...current, ...seeded]);
}

export default function WorkDashboard() {
  const navigate = useNavigate();

  // Require logged-in user (optional if you already have a route guard)
  useEffect(() => {
    if (!currentUserId()) navigate("/login", { replace: true });
  }, [navigate]);

  // Ensure profile is set to "work"
  useEffect(() => {
    localStorage.setItem("journal.selectedProfile", "work");
  }, []);

  // Seed demo work entries if none
  useEffect(() => {
    seedIfEmpty();
  }, []);

  const [nameQuery, setNameQuery] = useState("");
  const [dateQuery, setDateQuery] = useState("");       // YYYY-MM-DD
  const [categoryQuery, setCategoryQuery] = useState(""); // "", "A", "B", "C"
  const [entries, setEntries] = useState<Entry[]>([]);

  // Load entries for work profile
  useEffect(() => {
    const all = loadAllEntries();
    const mine = all.filter((e) => e.profile === "work");
    setEntries(mine.sort((a, b) => b.updatedAt.localeCompare(a.updatedAt)));
  }, []);

  // Filters: name contains, date exact (created OR updated), category equals
  const filtered = useMemo(() => {
    const nq = nameQuery.trim().toLowerCase();
    const dq = dateQuery;
    const cq = categoryQuery as Category | "";
    return entries.filter((e) => {
      const matchesName = !nq || e.name.toLowerCase().includes(nq);
      const createdDay = e.createdAt.slice(0, 10);
      const updatedDay = e.updatedAt.slice(0, 10);
      const matchesDate = !dq || createdDay === dq || updatedDay === dq;
      const matchesCategory = !cq || e.category === cq;
      return matchesName && matchesDate && matchesCategory;
    });
  }, [entries, nameQuery, dateQuery, categoryQuery]);

  function fmt(dt: string) {
    try {
      return new Date(dt).toLocaleString(undefined, {
        year: "numeric",
        month: "short",
        day: "2-digit",
        hour: "2-digit",
        minute: "2-digit",
      });
    } catch {
      return dt;
    }
  }

  return (
    <PaperBackground>
      {/* Top bar */}
      <header className="flex flex-col gap-3 md:flex-row md:items-center md:justify-between px-4 md:px-8 py-4">
        {/* LEFT: title + To-do / Growth */}
        <div className="flex items-center gap-3">
          <div>
            <h1 className="text-2xl font-semibold text-neutral-900">Work Dashboard</h1>
            <p className="text-sm text-neutral-600">Your work notes with categories & reports.</p>
          </div>
          <div className="flex items-center gap-2 ml-2">
            <Button variant="ghost" asChild className="bg-white/60 hover:bg-white">
              <Link to="/todo">To-do</Link>
            </Button>
            <Button variant="ghost" asChild className="bg-white/60 hover:bg-white">
              <Link to="/growth">Growth</Link>
            </Button>
          </div>
        </div>

        {/* RIGHT: create / switch / logout */}
        <div className="flex items-center gap-2">
          <Button asChild>
            <Link to="/new">Create New Entry</Link>
          </Button>
          <Button variant="ghost" asChild className="bg-white/60 hover:bg-white">
            <Link to="/profiles">Switch Profile</Link>
          </Button>
          <Button
            variant="ghost"
            onClick={() => {
              logout();
              navigate("/login", { replace: true });
            }}
          >
            Logout
          </Button>
        </div>
      </header>

      {/* Search bar */}
      <section className="px-4 md:px-8">
        <div className="grid gap-3 md:grid-cols-4">
          <div className="md:col-span-2">
            <label className="block text-sm font-medium text-neutral-800">Search by name</label>
            <input
              value={nameQuery}
              onChange={(e) => setNameQuery(e.target.value)}
              placeholder="e.g., Sprint planning notes"
              className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400 bg-white/80"
            />
          </div>
          <div>
            <label className="block text-sm font-medium text-neutral-800">Search by date</label>
            <input
              type="date"
              value={dateQuery}
              onChange={(e) => setDateQuery(e.target.value)}
              className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400 bg-white/80"
            />
          </div>
          <div>
            <label className="block text-sm font-medium text-neutral-800">Category</label>
            <select
              value={categoryQuery}
              onChange={(e) => setCategoryQuery(e.target.value)}
              className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 bg-white/80 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            >
              <option value="">All</option>
              <option value="A">A</option>
              <option value="B">B</option>
              <option value="C">C</option>
            </select>
          </div>
        </div>
      </section>

      {/* Entries table */}
      <section className="px-4 md:px-8 py-6">
        <div className="overflow-x-auto rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm">
          <table className="w-full text-left text-sm">
            <thead className="text-neutral-700">
              <tr className="border-b border-neutral-200">
                <th className="px-4 py-3 min-w-[220px]">Name</th>
                <th className="px-4 py-3 min-w-[160px]">Category</th>
                <th className="px-4 py-3 min-w-[180px]">Created at</th>
                <th className="px-4 py-3 min-w-[180px]">Last modified</th>
              </tr>
            </thead>
            <tbody>
              {filtered.length === 0 ? (
                <tr>
                  <td colSpan={4} className="px-4 py-8 text-center text-neutral-500">
                    No work entries found. Click <span className="font-medium">Create New Entry</span> to add one.
                  </td>
                </tr>
              ) : (
                filtered.map((e) => (
                  <tr key={e.id} className="border-t border-neutral-200 hover:bg-neutral-50/60">
                    <td className="px-4 py-3">
                      <button
                        className="text-neutral-900 hover:underline underline-offset-2"
                        onClick={() => navigate(`/entry/${e.id}`)}
                        title="Open entry"
                      >
                        {e.name}
                      </button>
                    </td>
                    <td className="px-4 py-3 text-neutral-700">{e.category ?? "—"}</td>
                    <td className="px-4 py-3 text-neutral-700">{fmt(e.createdAt)}</td>
                    <td className="px-4 py-3 text-neutral-700">{fmt(e.updatedAt)}</td>
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>
      </section>

      {/* Floating Generate button */}
      <Button
        asChild
        className="fixed bottom-6 right-6 h-12 px-6 rounded-full shadow-lg"
        variant="default"
        title="Generate report"
      >
        <Link to="/reports">Generate</Link>
      </Button>
    </PaperBackground>
  );
}



import { useEffect, useMemo, useState } from "react";
import { useNavigate, useSearchParams } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { getCurrentUser } from "@/lib/auth";

type Category = "A" | "B" | "C";

type Entry = {
  id: string;
  name: string;        // title
  body: string;
  createdAt: string;   // ISO
  updatedAt: string;   // ISO
  profile: "personal" | "work";
  category?: Category; // only for work
};

function currentUserId(): string | null {
  const u = getCurrentUser();
  return u ? u.id : null;
}
function userEntriesKey(uid: string) {
  return `journal.entries.${uid}`;
}
function loadAll(): Entry[] {
  const uid = currentUserId();
  if (!uid) return [];
  try {
    const raw = localStorage.getItem(userEntriesKey(uid));
    return raw ? (JSON.parse(raw) as Entry[]) : [];
  } catch {
    return [];
  }
}
function saveAll(entries: Entry[]) {
  const uid = currentUserId();
  if (!uid) return;
  localStorage.setItem(userEntriesKey(uid), JSON.stringify(entries));
}

export default function NewEntry() {
  const navigate = useNavigate();
  const [params] = useSearchParams();
  const editingId = params.get("id") || null;

  const selectedProfile = useMemo<"personal" | "work">(() => {
    const p = localStorage.getItem("journal.selectedProfile");
    return p === "work" ? "work" : "personal";
  }, []);

  const [title, setTitle] = useState("");
  const [body, setBody] = useState("");
  const [category, setCategory] = useState<Category | "">("");
  const [submitErr, setSubmitErr] = useState<string | null>(null);
  const [saving, setSaving] = useState(false);

  useEffect(() => {
    if (!currentUserId()) navigate("/login", { replace: true });
  }, [navigate]);

  // Load existing entry if editing
  useEffect(() => {
    if (!editingId) return;
    const all = loadAll();
    const found = all.find((e) => e.id === editingId);
    if (found) {
      setTitle(found.name);
      setBody(found.body ?? "");
      setCategory((found.category as Category) ?? "");
    } else {
      navigate("/dashboard", { replace: true });
    }
  }, [editingId, navigate]);

  function validate() {
    if (!title.trim()) {
      setSubmitErr("Title is required.");
      return false;
    }
    if (!body.trim()) {
      setSubmitErr("Body is required.");
      return false;
    }
    if (selectedProfile === "work" && !category) {
      setSubmitErr("Category is required for work entries.");
      return false;
    }
    setSubmitErr(null);
    return true;
  }

  async function onSave(e: React.FormEvent) {
    e.preventDefault();
    if (!validate()) return;
    setSaving(true);
    try {
      const now = new Date().toISOString();
      const all = loadAll();

      if (editingId) {
        const idx = all.findIndex((e) => e.id === editingId);
        if (idx >= 0) {
          const prev = all[idx];
          all[idx] = {
            ...prev,
            name: title.trim(),
            body: body.trim(),
            updatedAt: now,
            // if it’s a work entry, preserve/assign category
            category: prev.profile === "work" ? (category || prev.category) : undefined,
          };
          saveAll(all);
        }
      } else {
        const entry: Entry = {
          id: crypto.randomUUID(),
          name: title.trim(),
          body: body.trim(),
          createdAt: now,
          updatedAt: now,
          profile: selectedProfile,
          category: selectedProfile === "work" ? (category as Category) : undefined,
        };
        all.push(entry);
        saveAll(all);
      }

      // Route back to the correct dashboard
      navigate(selectedProfile === "work" ? "/dashboard-work" : "/dashboard", { replace: true });
    } finally {
      setSaving(false);
    }
  }

  return (
    <PaperBackground>
      <form onSubmit={onSave} className="min-h-screen w-full px-4 md:px-8 py-6">
        {/* Top bar */}
        <div className="mb-4 flex items-center justify-between gap-3">
          <div>
            <h1 className="text-2xl font-semibold text-neutral-900">
              {editingId ? "Edit Entry" : "New Entry"}
            </h1>
            <p className="text-sm text-neutral-600">
              Profile: <span className="font-medium capitalize">{selectedProfile}</span>
            </p>
          </div>
          <div className="flex items-center gap-2">
            <Button type="button" variant="ghost" onClick={() => navigate(-1)}>
              Cancel
            </Button>
            <Button type="submit" disabled={saving} aria-disabled={saving}>
              {saving ? "Saving…" : "Save"}
            </Button>
          </div>
        </div>

        {submitErr && (
          <div className="mb-4 rounded-md border border-red-200 bg-red-50 px-3 py-2 text-sm text-red-700">
            {submitErr}
          </div>
        )}

        {/* Title */}
        <div className="mb-3">
          <label className="block text-sm font-medium text-neutral-800">Title</label>
          <input
            className="mt-1 w-full rounded-md border border-neutral-300 bg-white/80 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            placeholder="e.g., Client call recap"
          />
        </div>

        {/* Body */}
        <div className="mb-3">
          <label className="block text-sm font-medium text-neutral-800">Body</label>
          <textarea
            className="mt-1 w-full min-h-[320px] rounded-md border border-neutral-300 bg-white/80 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            value={body}
            onChange={(e) => setBody(e.target.value)}
            placeholder="Write your entry…"
          />
        </div>

        {/* Category (only for work) */}
        {selectedProfile === "work" && (
          <div className="mb-3">
            <label className="block text-sm font-medium text-neutral-800">Category</label>
            <select
              value={category}
              onChange={(e) => setCategory(e.target.value as Category | "")}
              className="mt-1 w-full rounded-md border border-neutral-300 bg-white/80 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            >
              <option value="">Select a category…</option>
              <option value="A">A</option>
              <option value="B">B</option>
              <option value="C">C</option>
            </select>
          </div>
        )}
      </form>
    </PaperBackground>
  );
}



import { useEffect, useMemo, useState } from "react";
import { useNavigate, useParams, Link } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import ConfirmDialog from "@/components/ConfirmDialog";
import { getCurrentUser } from "@/lib/auth";

type Category = "A" | "B" | "C";
type Entry = {
  id: string;
  name: string;
  body: string;
  createdAt: string;   // ISO
  updatedAt: string;   // ISO
  profile: "personal" | "work";
  category?: Category;
};

function currentUserId(): string | null {
  const u = getCurrentUser();
  return u ? u.id : null;
}
function userEntriesKey(uid: string) {
  return `journal.entries.${uid}`;
}
function loadAll(): Entry[] {
  const uid = currentUserId();
  if (!uid) return [];
  try {
    const raw = localStorage.getItem(userEntriesKey(uid));
    return raw ? (JSON.parse(raw) as Entry[]) : [];
  } catch {
    return [];
  }
}
function saveAll(entries: Entry[]) {
  const uid = currentUserId();
  if (!uid) return;
  localStorage.setItem(userEntriesKey(uid), JSON.stringify(entries));
}

export default function EntryView() {
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  const [entry, setEntry] = useState<Entry | null>(null);
  const [confirmOpen, setConfirmOpen] = useState(false);

  const selectedProfile = useMemo<"personal" | "work">(() => {
    const p = localStorage.getItem("journal.selectedProfile");
    return p === "work" ? "work" : "personal";
  }, []);

  useEffect(() => {
    if (!currentUserId()) {
      navigate("/login", { replace: true });
      return;
    }
    const all = loadAll();
    const found = all.find((e) => e.id === id!);
    if (!found) {
      navigate("/dashboard", { replace: true });
      return;
    }
    setEntry(found);
  }, [id, navigate]);

  function fmt(dt: string) {
    try {
      return new Date(dt).toLocaleString(undefined, {
        year: "numeric",
        month: "short",
        day: "2-digit",
        hour: "2-digit",
        minute: "2-digit",
      });
    } catch {
      return dt;
    }
  }

  function requestDelete() {
    setConfirmOpen(true);
  }
  function confirmDelete() {
    if (!entry) return;
    const all = loadAll().filter((e) => e.id !== entry.id);
    saveAll(all);
    const backTo = entry.profile === "work" ? "/dashboard-work" : "/dashboard";
    navigate(backTo, { replace: true });
  }

  if (!entry) return null;

  const profileMismatch = entry.profile !== selectedProfile;

  return (
    <PaperBackground>
      <div className="min-h-screen w-full px-4 md:px-8 py-6">
        {/* Top bar */}
        <div className="mb-4 flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
          <div>
            <h1 className="text-2xl font-semibold text-neutral-900">{entry.name}</h1>
            <p className="text-sm text-neutral-600">
              Created: <span className="font-medium">{fmt(entry.createdAt)}</span>{" "}
              · Last modified: <span className="font-medium">{fmt(entry.updatedAt)}</span>{" "}
              · Profile: <span className="font-medium capitalize">{entry.profile}</span>
              {entry.profile === "work" && entry.category && (
                <>
                  {" "}
                  · Category: <span className="font-medium">{entry.category}</span>
                </>
              )}
            </p>
            {profileMismatch && (
              <p className="mt-1 text-xs text-amber-700 bg-amber-50 border border-amber-200 inline-block px-2 py-1 rounded">
                Viewing a {entry.profile} entry while {selectedProfile} is selected.
              </p>
            )}
          </div>
          <div className="flex items-center gap-2">
            <Button asChild variant="ghost" className="bg-white/60 hover:bg-white">
              <Link to={entry.profile === "work" ? "/dashboard-work" : "/dashboard"}>Back</Link>
            </Button>
            <Button asChild variant="outline">
              <Link to={`/new?id=${entry.id}`}>Edit</Link>
            </Button>
            <Button onClick={requestDelete}>Delete</Button>
          </div>
        </div>

        {/* Content */}
        <article className="rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm p-4">
          <div className="whitespace-pre-wrap text-[15px] leading-7 text-neutral-900">
            {entry.body}
          </div>
        </article>
      </div>

      <ConfirmDialog
        open={confirmOpen}
        title={`Delete “${entry.name}”?`}
        description="This action cannot be undone. The entry will be permanently removed."
        confirmText="Delete"
        cancelText="Cancel"
        onConfirm={confirmDelete}
        onCancel={() => setConfirmOpen(false)}
      />
    </PaperBackground>
  );
}
