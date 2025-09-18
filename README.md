import { motion } from "framer-motion";
import { PenLine } from "lucide-react";
import "@fontsource/pacifico";

export default function App() {
  return (
    <div className="min-h-screen grid place-items-center bg-white">
      <motion.div
        initial={{ opacity: 0, y: 8 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.4 }}
        className="text-center"
      >
        <h1 style={{ fontFamily: "'Pacifico', cursive" }} className="text-5xl text-neutral-900">
          Journal
        </h1>
        <div className="mt-3 flex items-center justify-center gap-2 text-neutral-700">
          <PenLine className="w-6 h-6" />
          <span>Framer + Lucide working ✅</span>
        </div>
      </motion.div>
    </div>
  );
}
