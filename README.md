async function onSubmit(e: React.FormEvent) {
  e.preventDefault();
  setSubmitErr(null);
  if (!validate()) return;

  setLoading(true);
  try {
    login(email.trim(), password);
    // after login, go choose a profile
    navigate("/profiles", { replace: true });
  } catch (err: any) {
    setSubmitErr(err?.message || "Invalid credentials.");
  } finally {
    setLoading(false);
  }
}
