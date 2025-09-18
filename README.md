import { Routes, Route, Navigate } from "react-router-dom";
import Landing from "@/pages/Landing";
import Signup from "@/pages/Auth/Signup";

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Landing />} />
      <Route path="/signup" element={<Signup />} />
      {/* optional placeholder for now */}
      <Route path="/login" element={<div style={{padding:24}}>Login page coming soon</div>} />
      <Route path="*" element={<Navigate to="/" replace />} />
    </Routes>
  );
}


import { useEffect, useRef, useState, useMemo } from "react";
import { motion } from "framer-motion";
import { Button } from "@/components/ui/button";
import { PenLine } from "lucide-react";
import { Link } from "react-router-dom";
import "@fontsource/pacifico";

export default function Landing() {
  const textRef = useRef<SVGTextElement | null>(null);
  const [dash, setDash] = useState(0);
  const [ready, setReady] = useState(false);

  useEffect(() => {
    let cancelled = false;
    (async () => {
      // Wait for the custom font before measuring text length
      // @ts-expect-error
      if (document.fonts?.ready) await document.fonts.ready;
      if (cancelled) return;
      if (textRef.current) {
        const L = Math.ceil(textRef.current.getComputedTextLength());
        setDash(L);
        setReady(true);
      }
    })();
    return () => {
      cancelled = true;
    };
  }, []);

  const paperStyle = useMemo<React.CSSProperties>(
    () => ({
      backgroundColor: "#faf7f2",
      backgroundImage: [
        "radial-gradient(circle at 20% 30%, rgba(0,0,0,0.025) 0.5px, transparent 1px)",
        "radial-gradient(circle at 80% 70%, rgba(0,0,0,0.02) 0.5px, transparent 1px)",
        "repeating-linear-gradient(90deg, rgba(0,0,0,0.018) 0, rgba(0,0,0,0.018) 1px, transparent 2px, transparent 6px)",
        "radial-gradient(ellipse at center, rgba(255,255,255,0.9) 0%, rgba(0,0,0,0.04) 100%)",
      ].join(","),
      backgroundSize:
        "240px 240px, 260px 260px, 12px 100%, 100% 100%",
      backgroundBlendMode: "multiply,multiply,normal,normal",
    }),
    []
  );

  return (
    <div
      className="min-h-screen w-full relative overflow-hidden"
      style={paperStyle}
    >
      {/* Top-right nav buttons */}
      <nav className="absolute right-4 top-4 flex items-center gap-3">
        <Button
          asChild
          variant="ghost"
          className="backdrop-blur-sm bg-white/50 hover:bg-white/80"
        >
          <Link to="/login">Log in</Link>
        </Button>
        <Button asChild>
          <Link to="/signup">Sign up</Link>
        </Button>
      </nav>

      {/* Center animated wordmark */}
      <main className="min-h-screen grid place-items-center">
        <div className="relative w-full max-w-4xl mx-auto px-6">
          <svg
            className="w-full"
            viewBox="0 0 1200 300"
            role="img"
            aria-labelledby="journalTitle"
          >
            <title id="journalTitle">Journal</title>

            {/* Outline animation */}
            <motion.text
              key={dash}
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
                strokeDasharray: dash || 1,
              }}
              initial={{ strokeDashoffset: dash || 1 }}
              animate={{ strokeDashoffset: 0 }}
              transition={{
                duration: 3.6,
                ease: [0.65, 0, 0.35, 1],
                delay: 0.2,
              }}
            >
              Journal
            </motion.text>

            {/* Fill fade-in */}
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
              animate={{ opacity: ready ? 1 : 0 }}
              transition={{ delay: 3.6, duration: 0.5 }}
            >
              Journal
            </motion.text>
          </svg>

          {/* Pen accent */}
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
