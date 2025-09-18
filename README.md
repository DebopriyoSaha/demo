import { useNavigate } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { Briefcase, User } from "lucide-react";

function ProfileCard({
  title,
  description,
  icon,
  onSelect,
}: {
  title: "Personal" | "Work";
  description: string;
  icon: React.ReactNode;
  onSelect: () => void;
}) {
  return (
    <button
      onClick={onSelect}
      className="group w-full rounded-2xl border border-neutral-200 bg-white/70 backdrop-blur-sm p-5 text-left shadow-sm transition
                 hover:shadow-md focus:outline-none focus:ring-2 focus:ring-neutral-400"
    >
      <div className="flex items-center gap-4">
        <div className="rounded-xl border border-neutral-200 bg-white p-3">
          {icon}
        </div>
        <div className="flex-1">
          <div className="text-lg font-semibold text-neutral-900">{title}</div>
          <div className="mt-0.5 text-sm text-neutral-600">{description}</div>
        </div>
        <div className="opacity-0 translate-x-1 transition group-hover:opacity-100 group-hover:translate-x-0 text-neutral-500">
          ➜
        </div>
      </div>
    </button>
  );
}

export default function Profiles() {
  const navigate = useNavigate();

  function selectProfile(kind: "personal" | "work") {
    localStorage.setItem("journal.selectedProfile", kind);
    // go wherever you want next (e.g., dashboard)
    navigate("/dashboard", { replace: true });
  }

  return (
    <PaperBackground>
      <div className="min-h-screen w-full grid place-items-center px-4">
        <div className="w-full max-w-2xl">
          <div className="mb-6 text-center">
            <h1 className="text-2xl font-semibold text-neutral-900">Choose your profile</h1>
            <p className="text-sm text-neutral-600 mt-1">
              Pick where you want today’s journaling to live. You can switch later.
            </p>
          </div>

          <div className="grid gap-4 md:grid-cols-2">
            <ProfileCard
              title="Personal"
              description="Private notes, reflections, habits."
              icon={<User className="w-6 h-6 text-neutral-900" />}
              onSelect={() => selectProfile("personal")}
            />
            <ProfileCard
              title="Work"
              description="Meeting notes, tasks, quick updates."
              icon={<Briefcase className="w-6 h-6 text-neutral-900" />}
              onSelect={() => selectProfile("work")}
            />
          </div>

          <div className="mt-6 text-center">
            <Button variant="ghost" onClick={() => navigate(-1)}>
              Back
            </Button>
          </div>
        </div>
      </div>
    </PaperBackground>
  );
}
