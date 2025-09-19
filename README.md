import { useEffect, useRef } from "react";
import { Button } from "@/components/ui/button";

type Props = {
  open: boolean;
  title?: string;
  description?: string;
  confirmText?: string;
  cancelText?: string;
  onConfirm: () => void;
  onCancel: () => void;
};

export default function ConfirmDialog({
  open,
  title = "Are you sure?",
  description,
  confirmText = "Delete",
  cancelText = "Cancel",
  onConfirm,
  onCancel,
}: Props) {
  const dialogRef = useRef<HTMLDivElement | null>(null);

  // basic focus + ESC handling
  useEffect(() => {
    if (!open) return;
    const prev = document.activeElement as HTMLElement | null;
    dialogRef.current?.focus();

    function onKey(e: KeyboardEvent) {
      if (e.key === "Escape") onCancel();
    }
    document.addEventListener("keydown", onKey);
    return () => {
      document.removeEventListener("keydown", onKey);
      prev?.focus?.();
    };
  }, [open, onCancel]);

  if (!open) return null;

  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center"
      aria-modal="true"
      role="dialog"
      aria-labelledby="cd-title"
      aria-describedby={description ? "cd-desc" : undefined}
    >
      {/* overlay */}
      <div
        className="absolute inset-0 bg-black/20 backdrop-blur-[1px]"
        onClick={onCancel}
      />

      {/* paper card */}
      <div
        ref={dialogRef}
        tabIndex={-1}
        className="relative z-10 w-full max-w-md rounded-2xl border border-neutral-200 bg-white/80 backdrop-blur-sm shadow-lg p-5"
      >
        <h2 id="cd-title" className="text-lg font-semibold text-neutral-900">
          {title}
        </h2>
        {description && (
          <p id="cd-desc" className="mt-1 text-sm text-neutral-700">
            {description}
          </p>
        )}

        <div className="mt-5 flex items-center justify-end gap-2">
          <Button variant="ghost" onClick={onCancel}>
            {cancelText}
          </Button>
          <Button onClick={onConfirm}>{confirmText}</Button>
        </div>
      </div>
    </div>
  );
}
