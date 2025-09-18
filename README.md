import { PropsWithChildren, useMemo } from "react";

export default function PaperBackground({ children }: PropsWithChildren) {
  const paperStyle = useMemo<React.CSSProperties>(
    () => ({
      backgroundColor: "#faf7f2",
      backgroundImage: [
        "radial-gradient(circle at 20% 30%, rgba(0,0,0,0.025) 0.5px, transparent 1px)",
        "radial-gradient(circle at 80% 70%, rgba(0,0,0,0.02) 0.5px, transparent 1px)",
        "repeating-linear-gradient(90deg, rgba(0,0,0,0.018) 0, rgba(0,0,0,0.018) 1px, transparent 2px, transparent 6px)",
        "radial-gradient(ellipse at center, rgba(255,255,255,0.9) 0%, rgba(0,0,0,0.04) 100%)",
      ].join(","),
      backgroundSize: "240px 240px, 260px 260px, 12px 100%, 100% 100%",
      backgroundBlendMode: "multiply,multiply,normal,normal",
    }),
    []
  );

  return (
    <div className="min-h-screen w-full relative" style={paperStyle}>
      {children}
    </div>
  );
}
