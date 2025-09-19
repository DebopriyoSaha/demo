import { useEffect, useMemo, useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { logout } from "@/lib/auth";

type Entry = {
  id: string;
  name: string;
  body?: string;
  createdAt: string;    // ISO
  updatedAt: string;    // ISO
  profile: "personal" | "work";
};

const STORAGE_KEY = "journal.entries";

function loadAllEntries(): Entry[] {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    return raw ? (JSON.parse(raw) as Entry[]) : [];
  } catch {
    return [];
  }
}
function saveAllEntries(entries: Entry[]) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(entries));
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
    {
      id: crypto.randomUUID(),
      name: "Team sync notes",
      body: "Discussed sprint tasks and blockers.",
      createdAt: iso(daysAgo(1)),
      updatedAt: iso(now),
      profile: "work",
    },
  ];
  saveAllEntries(seeded);
}

// ---- Component ----
export default function Dashboard() {
  const navigate = useNavigate();

  // Ensure a profile is selected; default to "personal"
  useEffect(() => {
    const selected = localStorage.getItem("journal.selectedProfile");
    if (!selected) localStorage.setItem("journal.selectedProfile", "personal");
  }, []);

  useEffect(() => {
    seedIfEmpty();
  }, []);

  const [nameQuery, setNameQuery] = useState("");
  const [dateQuery, setDateQuery] = useState(""); // YYYY-MM-DD
  const [entries, setEntries] = useState<Entry[]>([]);

  useEffect(() => {
    const selected = (localStorage.getItem("journal.selectedProfile") as Entry["profile"]) || "personal";
    const all = loadAllEntries();
    const mine = all.filter((e) => e.profile === selected);
    setEntries(mine.sort((a, b) => b.updatedAt.localeCompare(a.updatedAt)));
  }, []);

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
