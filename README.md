import { useEffect, useMemo, useState } from "react";
import { useNavigate, useSearchParams } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";

type Entry = {
  id: string;
  name: string;        // title
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

export default function NewEntry() {
  const navigate = useNavigate();
  const [params] = useSearchParams();
  const editingId = params.get("id") || null;

  const selectedProfile = useMemo<"personal" | "work">(() => {
    const p = localStorage.getItem("journal.selectedProfile");
    return (p === "work" ? "work" : "personal");
  }, []);

  const [title, setTitle] = useState("");
  const [body, setBody] = useState("");
  const [submitErr, setSubmitErr] = useState<string | null>(null);
  const [saving, setSaving] = useState(false);

  // Load existing entry if editing
  useEffect(() => {
    if (!editingId) return;
    const all = loadAll();
    const found = all.find((e) => e.id === editingId);
    if (found) {
      setTitle(found.name);
      setBody(found.body ?? "");
    } else {
      // bad id → go back to dashboard
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
        // update existing
        const idx = all.findIndex((e) => e.id === editingId);
        if (idx >= 0) {
          const prev = all[idx];
          all[idx] = {
            ...prev,
            name: title.trim(),
            body: body.trim(),
            updatedAt: now,
            // keep existing profile; if you want to enforce current selection, uncomment next line
            // profile: selectedProfile,
          };
          saveAll(all);
        }
      } else {
        // create new
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
