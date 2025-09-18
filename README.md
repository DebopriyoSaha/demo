import { useEffect, useMemo, useRef, useState } from "react";
import { motion } from "framer-motion";
import { Button } from "@/components/ui/button";
import { PenLine } from "lucide-react";
import "@fontsource/pacifico";

export default function App() {
  const textRef = useRef<SVGTextElement | null>(null);
  const [dash, setDash] = useState<number>(0);
  const [ready, setReady] = useState(false);

  // Wait for fonts to load, then measure text accurately
  useEffect(() => {
    let cancelled = false;
    (async () => {
      try {
        // Wait for all document fonts, including @fontsource, to be ready
        if (document.fonts && document.fonts.ready) {
          await document.fonts.ready;
        } else {
          // small fallback delay if FontFaceSet isn't available
          await new Promise(r => setTimeout(r, 100));
        }
      } catch {}
      if (cancelled) return;
      if (textRef.current) {
        const L = Math.ceil(textRef.current.getComputedTextLength());
        setDash(L);
        setReady(true);
      }
    })();
    return () => { cancelled = true; };
  }, []);

  // Measure the SVG <text> to animate strokeDashoffset from length -> 0
  useEffect(() => {
    if (textRef.current) {
      const L = Math.ceil(textRef.current.getComputedTextLength());
      setDash(L);
    }
  }, []);

  // Paper-like background (pure CSS)
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
    <div className="min-h-screen w-full relative overflow-hidden" style={paperStyle}>
      {/* Top-right auth buttons */}
      <nav className="absolute right-4 top-4 flex items-center gap-3">
        <Button variant="ghost" className="backdrop-blur-sm bg-white/50 hover:bg-white/80 cursor-pointer" >
          Log in
        </Button>
        <Button>Sign up</Button>
      </nav>

      {/* Center: animated handwritten wordmark */}
      <main className="min-h-screen grid place-items-center">
        <div className="relative w-full max-w-4xl mx-auto px-6">
          <svg className="w-full" viewBox="0 0 1200 300" role="img" aria-labelledby="journalTitle">
            <title id="journalTitle">Journal</title>

            {/* Outline that draws */}
            <motion.text
              key={dash}                    // re-run animation after measuring with the real font
              ref={textRef}
              x="50%"
              y="55%"
              textAnchor="middle"
              style={{
                fontFamily: "'Pacifico', ui-serif, Georgia, serif",
                fontSize: 180,
                fill: "none",
                stroke: "#111827",
                strokeWidth: 2.2,
                strokeLinecap: "round",
                strokeLinejoin: "round",
                strokeDasharray: dash || 1, // avoid 0
              }}
              initial={{ strokeDashoffset: dash || 1 }}
              animate={{ strokeDashoffset: 0 }}
              transition={{ duration: 3.6, ease: [0.65, 0, 0.35, 1], delay: 0.2 }}
            >
              Journal
            </motion.text>

            {/* Filled text fades in after outline finishes */}
            <motion.text
              x="50%"
              y="55%"
              textAnchor="middle"
              style={{
                fontFamily: "'Pacifico', ui-serif, Georgia, serif",
                fontSize: 180,
                fill: "#111827",
              }}
              initial={{ opacity: 0 }}
              animate={{ opacity: ready ? 1 : 0 }}  // wait until outline measured
              transition={{ delay: 3.6, duration: 0.5 }}
            >
              Journal
            </motion.text>
          </svg>
          {/* Little pen accent pop near the end */}
          <motion.div
            className="absolute -right-2 top-1/2 -translate-y-1/2 select-none pointer-events-none"
            initial={{ opacity: 0, scale: 0.9 }}
            animate={{ opacity: 1, scale: 1 }}
            transition={{ delay: 3.4, duration: 0.4 }}
            aria-hidden
          >
            <PenLine className="w-8 h-8 text-neutral-900" />
          </motion.div>
        </div>
      </main>
    </div>
  );
}
