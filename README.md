import { useEffect, useMemo, useState } from "react";
import { useNavigate, useSearchParams } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { getCurrentUser } from "@/lib/auth";

type Entry = {
  id: string;
  name: string;        // title
  body: string;
  createdAt: string;   // ISO
  updatedAt: string;   // ISO
  profile: "personal" | "work";
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
  const [submitErr, setSubmitErr] = useState<string | null>(null);
  const [saving, setSaving] = useState(false);

  // If not logged in, bounce (optional; use your RequireAuth if you have it)
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
        };
        all.push(entry);
        saveAll(all);
      }

      navigate("/dashboard", { replace: true });
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
            placeholder="e.g., Morning reflections"
          />
        </div>

        {/* Body */}
        <div>
          <label className="block text-sm font-medium text-neutral-800">Body</label>
          <textarea
            className="mt-1 w-full min-h-[320px] rounded-md border border-neutral-300 bg-white/80 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
            value={body}
            onChange={(e) => setBody(e.target.value)}
            placeholder="Write your entry…"
          />
        </div>
      </form>
    </PaperBackground>
  );
}



import { useEffect, useMemo, useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { logout, getCurrentUser } from "@/lib/auth";

type Entry = {
  id: string;
  name: string;
  body?: string;
  createdAt: string;    // ISO
  updatedAt: string;    // ISO
  profile: "personal" | "work";
};

const LEGACY_KEY = "journal.entries";

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
/** one-time migration from legacy global key to this user's bucket */
function migrateLegacyIfAny() {
  const uid = currentUserId();
  if (!uid) return;
  const targetKey = userEntriesKey(uid);
  if (localStorage.getItem(targetKey)) return; // already migrated
  const legacy = localStorage.getItem(LEGACY_KEY);
  if (legacy) {
    localStorage.setItem(targetKey, legacy);
    // optional: localStorage.removeItem(LEGACY_KEY);
  }
}
function seedIfEmpty() {
  const current = loadAllEntries();
  if (current.length > 0) return;
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
      name: "Morning reflections",
      body: "Woke up early and felt productive.",
      createdAt: iso(daysAgo(3)),
      updatedAt: iso(daysAgo(2)),
      profile: "personal",
    },
    {
      id: crypto.randomUUID(),
      name: "Weekend recap",
      body: "Relaxed, went hiking, recharged.",
      createdAt: iso(daysAgo(7)),
      updatedAt: iso(daysAgo(6)),
      profile: "personal",
    },
  ];
  saveAllEntries(seeded);
}

export default function Dashboard() {
  const navigate = useNavigate();

  // Require a logged-in user (optional if you use a route guard)
  useEffect(() => {
    if (!currentUserId()) navigate("/login", { replace: true });
  }, [navigate]);

  // Ensure a profile is selected; default to "personal"
  useEffect(() => {
    const selected = localStorage.getItem("journal.selectedProfile");
    if (!selected) localStorage.setItem("journal.selectedProfile", "personal");
  }, []);

  // Per-user migration + seed
  useEffect(() => {
    migrateLegacyIfAny();
    seedIfEmpty();
  }, []);

  const [nameQuery, setNameQuery] = useState("");
  const [dateQuery, setDateQuery] = useState(""); // YYYY-MM-DD
  const [entries, setEntries] = useState<Entry[]>([]);

  // Load entries for the current profile
  useEffect(() => {
    const selected = (localStorage.getItem("journal.selectedProfile") as Entry["profile"]) || "personal";
    const all = loadAllEntries();
    const mine = all.filter((e) => e.profile === selected);
    setEntries(mine.sort((a, b) => b.updatedAt.localeCompare(a.updatedAt)));
  }, []);

  // Filtered rows
  const filtered = useMemo(() => {
    const nq = nameQuery.trim().toLowerCase();
    const dq = dateQuery;
    return entries.filter((e) => {
      const matchesName = !nq || e.name.toLowerCase().includes(nq);
      const createdDay = e.createdAt.slice(0, 10);
      const updatedDay = e.updatedAt.slice(0, 10);
      const matchesDate = !dq || createdDay === dq || updatedDay === dq;
      return matchesName && matchesDate;
    });
  }, [entries, nameQuery, dateQuery]);

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
        <div>
          <h1 className="text-2xl font-semibold text-neutral-900">Personal Dashboard</h1>
          <p className="text-sm text-neutral-600">Browse and filter your personal journal entries.</p>
        </div>
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
        <div className="grid gap-3 md:grid-cols-3">
          <div className="md:col-span-2">
            <label className="block text-sm font-medium text-neutral-800">Search by name</label>
            <input
              value={nameQuery}
              onChange={(e) => setNameQuery(e.target.value)}
              placeholder="e.g., Morning reflections"
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
        </div>
      </section>

      {/* Entries table */}
      <section className="px-4 md:px-8 py-6">
        <div className="overflow-x-auto rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm">
          <table className="w-full text-left text-sm">
            <thead className="text-neutral-700">
              <tr className="border-b border-neutral-200">
                <th className="px-4 py-3 min-w-[200px]">Name</th>
                <th className="px-4 py-3 min-w-[180px]">Created at</th>
                <th className="px-4 py-3 min-w-[180px]">Last modified</th>
              </tr>
            </thead>
            <tbody>
              {filtered.length === 0 ? (
                <tr>
                  <td colSpan={3} className="px-4 py-8 text-center text-neutral-500">
                    No entries found. Click <span className="font-medium">Create New Entry</span> to add one.
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
                    <td className="px-4 py-3 text-neutral-700">{fmt(e.createdAt)}</td>
                    <td className="px-4 py-3 text-neutral-700">{fmt(e.updatedAt)}</td>
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>
      </section>
    </PaperBackground>
  );
}



import { useEffect, useMemo, useState } from "react";
import { useNavigate, useParams, Link } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import ConfirmDialog from "@/components/ConfirmDialog";
import { getCurrentUser } from "@/lib/auth";

type Entry = {
  id: string;
  name: string;
  body: string;
  createdAt: string;   // ISO
  updatedAt: string;   // ISO
  profile: "personal" | "work";
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
    navigate("/dashboard", { replace: true });
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
            </p>
            {profileMismatch && (
              <p className="mt-1 text-xs text-amber-700 bg-amber-50 border border-amber-200 inline-block px-2 py-1 rounded">
                Viewing a {entry.profile} entry while {selectedProfile} is selected.
              </p>
            )}
          </div>
          <div className="flex items-center gap-2">
            <Button asChild variant="ghost" className="bg-white/60 hover:bg-white">
              <Link to="/dashboard">Back</Link>
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
