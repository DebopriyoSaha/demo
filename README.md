import { useState } from "react";
import { useNavigate, Link } from "react-router-dom";
import PaperBackground from "@/components/PaperBackground";
import { Button } from "@/components/ui/button";
import { Loader2 } from "lucide-react";

/** Fake API: succeeds unless email contains "error" */
async function fakeSignupApi(data: {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  phone: string;
}) {
  await new Promise((r) => setTimeout(r, 800));
  if (data.email.toLowerCase().includes("error")) {
    throw new Error("That email address is already in use.");
  }
  return { ok: true };
}

type FormState = {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  confirmPassword: string;
  phone: string;
};

export default function Signup() {
  const navigate = useNavigate();

  const [form, setForm] = useState<FormState>({
    firstName: "",
    lastName: "",
    email: "",
    password: "",
    confirmPassword: "",
    phone: "",
  });

  const [fieldErrors, setFieldErrors] = useState<Partial<Record<keyof FormState, string>>>({});
  const [submitError, setSubmitError] = useState<string | null>(null);
  const [submitting, setSubmitting] = useState(false);

  function onChange<K extends keyof FormState>(key: K, value: string) {
    setForm((f) => ({ ...f, [key]: value }));
    setFieldErrors((e) => ({ ...e, [key]: undefined })); // clear per-field error on change
    setSubmitError(null);
  }

  function validate(): boolean {
    const errors: Partial<Record<keyof FormState, string>> = {};

    if (!form.firstName.trim()) errors.firstName = "First name is required.";
    if (!form.lastName.trim()) errors.lastName = "Last name is required.";

    const emailOk = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email);
    if (!emailOk) errors.email = "Enter a valid email.";

    if (form.password.length < 8) errors.password = "Password must be at least 8 characters.";
    if (form.confirmPassword !== form.password) errors.confirmPassword = "Passwords do not match.";

    // very light phone validation (allows digits, spaces, +, -, parentheses). Adjust as needed.
    const phoneOk = /^[\d\s+\-()]{7,}$/.test(form.phone);
    if (!phoneOk) errors.phone = "Enter a valid phone number.";

    setFieldErrors(errors);
    return Object.keys(errors).length === 0;
  }

  async function onSubmit(e: React.FormEvent) {
    e.preventDefault();
    setSubmitError(null);

    if (!validate()) return;

    setSubmitting(true);
    try {
      await fakeSignupApi({
        firstName: form.firstName.trim(),
        lastName: form.lastName.trim(),
        email: form.email.trim(),
        password: form.password,
        phone: form.phone.trim(),
      });
      // success -> go back to landing
      navigate("/", { replace: true });
    } catch (err: any) {
      setSubmitError(err?.message || "Something went wrong. Please try again.");
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <PaperBackground>
      <div className="min-h-screen w-full grid place-items-center px-4">
        <form
          onSubmit={onSubmit}
          className="w-full max-w-md bg-white/70 backdrop-blur-sm rounded-xl p-6 border border-neutral-200 shadow-sm"
        >
          <h1 className="text-2xl font-semibold text-neutral-900">Create your account</h1>
          <p className="text-sm text-neutral-600 mb-4">Start journaling in seconds.</p>

          {submitError && (
            <div className="mb-4 rounded-md border border-red-200 bg-red-50 px-3 py-2 text-sm text-red-700">
              {submitError}
            </div>
          )}

          <div className="grid gap-3">
            <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
              <div>
                <label className="block text-sm font-medium text-neutral-800">First name</label>
                <input
                  className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
                  value={form.firstName}
                  onChange={(e) => onChange("firstName", e.target.value)}
                  autoComplete="given-name"
                />
                {fieldErrors.firstName && (
                  <p className="mt-1 text-xs text-red-600">{fieldErrors.firstName}</p>
                )}
              </div>

              <div>
                <label className="block text-sm font-medium text-neutral-800">Last name</label>
                <input
                  className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
                  value={form.lastName}
                  onChange={(e) => onChange("lastName", e.target.value)}
                  autoComplete="family-name"
                />
                {fieldErrors.lastName && (
                  <p className="mt-1 text-xs text-red-600">{fieldErrors.lastName}</p>
                )}
              </div>
            </div>

            <div>
              <label className="block text-sm font-medium text-neutral-800">Email</label>
              <input
                className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
                type="email"
                value={form.email}
                onChange={(e) => onChange("email", e.target.value)}
                autoComplete="email"
              />
              {fieldErrors.email && (
                <p className="mt-1 text-xs text-red-600">{fieldErrors.email}</p>
              )}
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
              <div>
                <label className="block text-sm font-medium text-neutral-800">Password</label>
                <input
                  className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
                  type="password"
                  value={form.password}
                  onChange={(e) => onChange("password", e.target.value)}
                  autoComplete="new-password"
                />
                {fieldErrors.password && (
                  <p className="mt-1 text-xs text-red-600">{fieldErrors.password}</p>
                )}
              </div>

              <div>
                <label className="block text-sm font-medium text-neutral-800">Confirm password</label>
                <input
                  className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
                  type="password"
                  value={form.confirmPassword}
                  onChange={(e) => onChange("confirmPassword", e.target.value)}
                  autoComplete="new-password"
                />
                {fieldErrors.confirmPassword && (
                  <p className="mt-1 text-xs text-red-600">{fieldErrors.confirmPassword}</p>
                )}
              </div>
            </div>

            <div>
              <label className="block text-sm font-medium text-neutral-800">Phone number</label>
              <input
                className="mt-1 w-full rounded-md border border-neutral-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-neutral-400"
                inputMode="tel"
                value={form.phone}
                onChange={(e) => onChange("phone", e.target.value)}
                autoComplete="tel"
                placeholder="+1 (555) 555-5555"
              />
              {fieldErrors.phone && (
                <p className="mt-1 text-xs text-red-600">{fieldErrors.phone}</p>
              )}
            </div>
          </div>

          <Button
            type="submit"
            className="mt-4 w-full"
            disabled={submitting}
            aria-disabled={submitting}
          >
            {submitting ? (
              <span className="inline-flex items-center gap-2">
                <Loader2 className="h-4 w-4 animate-spin" />
                Creating account…
              </span>
            ) : (
              "Create account"
            )}
          </Button>

          <p className="mt-3 text-sm text-neutral-600">
            Already have an account?{" "}
            <Link className="underline underline-offset-2" to="/login">
              Log in
            </Link>
          </p>
        </form>
      </div>
    </PaperBackground>
  );
}
