async function onSubmit(e: React.FormEvent) {
  e.preventDefault();
  setSubmitError(null);

  if (!validate()) return;

  setSubmitting(true);
  try {
    signup({
      firstName: form.firstName.trim(),
      lastName: form.lastName.trim(),
      email: form.email.trim(),
      password: form.password,
      phone: form.phone.trim(),
    });
    // after successful signup, go choose a profile
    navigate("/profiles", { replace: true });
  } catch (err: any) {
    setSubmitError(err?.message || "Something went wrong. Please try again.");
  } finally {
    setSubmitting(false);
  }
}
