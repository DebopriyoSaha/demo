import { useState } from "react";
import { useNavigate, Link } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { Loader2 } from "lucide-react";

/** Fake API: fails if email contains "error" */
async function fakeLoginApi(data: { email: string; password: string }) {
  await new Promise((r) => setTimeout(r, 600));
  if (data.email.toLowerCase().includes("error")) {
    throw new Error("Invalid credentials. Please try again.");
  }
  return { ok: true };
}

export default function Login() {
  const navigate = useNavigate();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [emailErr, setEmailErr] = useState<string | null>(null);
  const [passwordErr, setPasswordErr] = useState<string | null>(null);
  const [submitErr, setSubmitErr] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  function validate() {
    let ok = true;
    setEmailErr(null);
    setPasswordErr(null);

    const emailOk = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    if (!emailOk) {
      setEmailErr("Enter a valid email.");
      ok = false;
    }
    if (password.length < 8) {
      setPasswordErr("Password must be at least 8 characters.");
      ok = false;
    }
    return ok;
  }

  async function onSubmit(e: React.FormEvent) {
    e.preventDefault();
    setSubmitErr(null);
    if (!validate()) return;

    setLoading(true);
    try {
      await fakeLoginApi({ email: email.trim(), password });
      // success → back to landing (adjust to /dashboard later if you add it)
      navigate("/", { replace: true });
    } catch (err: any) {
      setSubmitErr(err?.message || "Something went wrong. Please try again.");
    } finally {
      setLoading(false);
    }
  }

  return (
    <PaperBackground>
      <div className="min-h-screen w-full grid place-items-center px-4">
        <form
          onSubmit={onSubmit}
          className="w-full max-w-md bg-white/70 backdrop-blur-sm rounded-xl p-6 border border-neutral-200 shadow-sm"
        >
          <h1 className="text-2xl font-semibold text-neutral-900">Log in</h1>
          <p className="text-sm text-neutral-600 mb-4">Welcome back.</p>

          {submitErr && (
            <div className="mb-4 rounded-md border border-red-200 bg-red-50 px-3 py-2 text-sm text-red-700">
              {submitErr}
            </div>
          )}

          <div className="grid gap-3">
            <div>
              <label className="block text-sm font-medium text-neutral-800">Email</label>
              <input
                className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
                type="email"
                value={email}
                onChange={(e) => {
                  setEmail(e.target.value);
                  setEmailErr(null);
                  setSubmitErr(null);
                }}
                autoComplete="email"
              />
              {emailErr && <p className="mt-1 text-xs text-red-600">{emailErr}</p>}
            </div>

            <div>
              <label className="block text-sm font-medium text-neutral-800">Password</label>
              <input
                className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
                type="password"
                value={password}
                onChange={(e) => {
                  setPassword(e.target.value);
                  setPasswordErr(null);
                  setSubmitErr(null);
                }}
                autoComplete="current-password"
              />
              {passwordErr && <p className="mt-1 text-xs text-red-600">{passwordErr}</p>}
            </div>
          </div>

          <Button type="submit" className="mt-4 w-full" disabled={loading} aria-disabled={loading}>
            {loading ? (
              <span className="inline-flex items-center gap-2">
                <Loader2 className="h-4 w-4 animate-spin" />
                Logging in…
              </span>
            ) : (
              "Log in"
            )}
          </Button>

          <p className="mt-3 text-sm text-neutral-600">
            Don’t have an account?{" "}
            <Link className="underline underline-offset-2" to="/signup">
              Sign up
            </Link>
          </p>
        </form>
      </div>
    </PaperBackground>
  );
}
