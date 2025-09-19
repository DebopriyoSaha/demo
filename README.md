import { useEffect, useMemo, useState } from "react";
import { useNavigate, useParams, Link } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";

type Entry = {
  id: string;
  name: string;
  body: string;
  createdAt: string;   // ISO
  updatedAt: string;   // ISO
  profile: "personal" | "work";
};

const STORAGE_KEY = "journal.entries";

function loadAll(): Entry[] {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    return raw ? (JSON.parse(raw) as Entry[]) : [];
  } catch {
    return [];
  }
}
function saveAll(entries: Entry[]) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(entries));
}

export default function EntryView() {
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  const [entry, setEntry] = useState<Entry | null>(null);

  const selectedProfile = useMemo<"personal" | "work">(() => {
    const p = localStorage.getItem("journal.selectedProfile");
    return p === "work" ? "work" : "personal";
  }, []);

  useEffect(() => {
    const all = loadAll();
    const found = all.find((e) => e.id === id!);
    if (!found) {
      // not found → go back to dashboard
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

  function onDelete() {
    if (!entry) return;
    const ok = confirm(`Delete “${entry.name}”? This cannot be undone.`);
    if (!ok) return;
    const all = loadAll().filter((e) => e.id !== entry.id);
    saveAll(all);
    navigate("/dashboard", { replace: true });
  }

  if (!entry) return null;

  // Optional: if someone opens a work entry while they’re on personal profile,
  // you can hint at that (but still show the content).
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
            <Button variant="default" onClick={onDelete}>
              Delete
            </Button>
          </div>
        </div>

        {/* Content */}
        <article className="rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm p-4">
          <div className="whitespace-pre-wrap text-[15px] leading-7 text-neutral-900">
            {entry.body}
          </div>
        </article>
      </div>
    </PaperBackground>
  );
}
