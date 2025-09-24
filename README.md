// SlimBak Single-File App (index.tsx)
// -------------------------------------------------------------
// This replaces the previous multi-file repo to fix the error:
// "SyntaxError: /index.tsx: Missing semicolon. (9:8)"
// The error happened because JSON/other files were pasted into a TSX file.
// This is a clean **single TSX file** you can run in Vite/CRA/Codesandbox.
// - If using **Vite React TS**: keep this as src/main.tsx or src/index.tsx.
// - If using **Next.js**: put the component body as pages/index.tsx and
//   REMOVE the createRoot block at the bottom (Next renders for you).
// - No external UI libs required. All semicolons and types are correct.
// -------------------------------------------------------------

import React, { useMemo, useState } from "react";
import { createRoot } from "react-dom/client";

// -------------------- Types & Utilities --------------------
type Currency = "EUR" | "USD";

function clampQty(n: number): number {
  return Number.isFinite(n) && n > 0 ? Math.floor(n) : 1;
}

function priceByCurrency(currency: Currency): number {
  return currency === "USD" ? 32.99 : 29.99; // SlimBak towel base price
}

function computeTotal(qty: number, currency: Currency): number {
  return +(priceByCurrency(currency) * clampQty(qty)).toFixed(2);
}

// Pure util for testing locale → currency without accessing navigator
export function currencyFromLang(lang: string): Currency {
  const code = (lang || "").toLowerCase();
  if (code.includes("en-us") || code.endsWith("-us")) return "USD";
  return "EUR";
}

function detectCurrency(): Currency {
  try {
    // Try navigator.language safely (browser only)
    const lang = typeof navigator !== "undefined" ? navigator.language : "";
    return currencyFromLang(lang);
  } catch {
    return "EUR";
  }
}

// -------------------- Mini Test Cases --------------------
// We add a few runtime sanity checks (they print to console). Never modify
// existing tests unless they are clearly wrong; here we only add new ones.
(function runMiniTests() {
  // computeTotal tests
  console.assert(computeTotal(1, "EUR") === 29.99, "EUR x1 should be 29.99");
  console.assert(computeTotal(2, "EUR") === 59.98, "EUR x2 should be 59.98");
  console.assert(computeTotal(1, "USD") === 32.99, "USD x1 should be 32.99");
  console.assert(computeTotal(3, "USD") === +(32.99 * 3).toFixed(2), "USD x3 total");

  // currencyFromLang tests
  console.assert(currencyFromLang("en-US") === "USD", "en-US → USD");
  console.assert(currencyFromLang("de-DE") === "EUR", "de-DE → EUR");
  console.assert(currencyFromLang("") === "EUR", "empty → EUR");
})();

// -------------------- Simple Styles --------------------
const styles: Record<string, React.CSSProperties> = {
  section: { padding: "56px 16px" },
  container: { maxWidth: 1100, margin: "0 auto" },
  h1: { fontSize: 42, fontWeight: 800, margin: "0 0 12px" },
  h2: { fontSize: 32, fontWeight: 800, margin: "0 0 16px" },
  p: { opacity: 0.9, lineHeight: 1.6 },
  btn: {
    borderRadius: 9999,
    background: "#2563eb",
    color: "#fff",
    padding: "12px 22px",
    border: 0,
    cursor: "pointer",
    fontWeight: 600
  },
  btnDark: { borderRadius: 9999, background: "#1f2937", color: "#fff", padding: "12px 22px", border: 0, cursor: "pointer", fontWeight: 600 },
  card: { borderRadius: 16, border: "1px solid #2a2a2a", padding: 24, background: "#0b0b0b", color: "#fff" },
  input: { padding: "10px 14px", borderRadius: 12, border: "1px solid #d1d5db" },
  hero: {
    minHeight: "78vh",
    display: "flex",
    alignItems: "center",
    justifyContent: "center",
    textAlign: "center",
    background: "linear-gradient(120deg,#000,#0b0b12)",
    color: "#fff",
    position: "relative"
  },
  footer: { borderTop: "1px solid #1f2937", padding: "24px 16px", textAlign: "center", color: "#9ca3af" },
  link: { color: "#9ca3af", textDecoration: "none", margin: "0 10px" }
};

// -------------------- App --------------------
function App(): JSX.Element {
  const [currency, setCurrency] = useState<Currency>(detectCurrency());
  const [qty, setQty] = useState<number>(1);
  const [showPolicy, setShowPolicy] = useState<null | "privacy" | "terms" | "refund" | "shipping">(null);

  const total = useMemo(() => computeTotal(qty, currency), [qty, currency]);

  async function startCheckout(): Promise<void> {
    // Attempt POST to /api/checkout (if you wire a backend). If it fails,
    // fallback to an alert so this file runs anywhere.
    try {
      const res = await fetch("/api/checkout", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ currency, quantity: clampQty(qty) })
      });
      if (res.ok) {
        const data = await res.json();
        if (data?.url) {
          window.location.href = data.url as string;
          return;
        }
      }
      // Fallback
      alert("Checkout endpoint not configured. Deploy backend or set Stripe link.");
    } catch {
      alert("Checkout not available in this preview. Configure backend to proceed.");
    }
  }

  function openPolicy(which: typeof showPolicy): void { setShowPolicy(which); }
  function closePolicy(): void { setShowPolicy(null); }

  return (
    <div style={{ background: "#000", color: "#fff", fontFamily: "Inter, system-ui, Arial, sans-serif" }}>
      {/* Hero */}
      <section style={styles.hero}>
        <div style={{ ...styles.container, padding: 16 }}>
          <h1 style={styles.h1}>SlimBak – Your Fitness Style, Made in Tunisia</h1>
          <p style={{ ...styles.p, margin: "0 0 18px" }}>
            The first magnetic gym towel designed for performance, hygiene, and style.
          </p>
          <div style={{ display: "flex", gap: 12, justifyContent: "center", flexWrap: "wrap" }}>
            <button style={styles.btn} onClick={startCheckout}>Shop Now</button>
            <button style={styles.btnDark} onClick={() => {
              const el = document.getElementById("cart");
              if (el) el.scrollIntoView({ behavior: "smooth" });
            }}>Aperçu</button>
          </div>
          <div style={{ marginTop: 14, display: "flex", gap: 8, justifyContent: "center", alignItems: "center" }}>
            <span style={{ color: "#d1d5db", fontSize: 14 }}>Currency:</span>
            <select
              value={currency}
              onChange={(e) => setCurrency(e.target.value as Currency)}
              style={{ background: "#111827", color: "#fff", border: "1px solid #374151", borderRadius: 12, padding: "6px 10px" }}
            >
              <option value="EUR">EUR (€)</option>
              <option value="USD">USD ($)</option>
            </select>
          </div>
        </div>
      </section>

      {/* Features */}
      <section style={styles.section}>
        <div style={styles.container}>
          <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(220px,1fr))", gap: 18 }}>
            {[{
              title: "Magnetic Clip",
              desc: "Attach your towel anywhere, hands-free."
            },{
              title: "Premium Fabric",
              desc: "Soft, absorbent, and long-lasting."
            },{
              title: "Luxury Design",
              desc: "A fitness essential that matches your style."
            }].map((f) => (
              <div key={f.title} style={styles.card}>
                <h3 style={{ fontSize: 18, fontWeight: 700, margin: "0 0 6px" }}>{f.title}</h3>
                <p style={styles.p}>{f.desc}</p>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* Product Showcase */}
      <section style={{ ...styles.section, background: "#fff", color: "#000", textAlign: "center" }}>
        <div style={styles.container}>
          <h2 style={styles.h2}>Meet SlimBak</h2>
          <div style={{ height: 280, background: "#eef2ff", borderRadius: 16, margin: "0 auto 16px", maxWidth: 700 }} />
          <p style={{ ...styles.p, margin: "0 auto 16px", maxWidth: 720 }}>
            Born in Tunisia, SlimBak is more than a towel – it’s a fitness companion. Designed with athletes in mind, it brings style,
            hygiene, and performance together.
          </p>
          <button style={{ ...styles.btn, color: "#fff" }} onClick={startCheckout}>Buy Now</button>
        </div>
      </section>

      {/* Shop Grid */}
      <section style={{ ...styles.section, background: "#0f0f12", textAlign: "center" }}>
        <div style={styles.container}>
          <h2 style={styles.h2}>Shop SlimBak</h2>
          <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(260px,1fr))", gap: 16 }}>
            {[1,2,3].map(i => (
              <div key={i} style={styles.card}>
                <div style={{ height: 180, background: "#111827", borderRadius: 12, marginBottom: 12 }} />
                <h3 style={{ fontSize: 18, fontWeight: 700, margin: "0 0 6px" }}>SlimBak Towel {i}</h3>
                <p style={{ ...styles.p, marginBottom: 12 }}>High-quality gym towel with magnetic clip and luxury design.</p>
                <div style={{ display: "flex", gap: 10, justifyContent: "center", alignItems: "center", marginBottom: 12 }}>
                  <label style={{ fontSize: 13, opacity: 0.9 }}>Qty</label>
                  <input
                    type="number"
                    value={qty}
                    min={1}
                    onChange={(e) => setQty(clampQty(Number(e.target.value)))}
                    style={{ ...styles.input, width: 70 }}
                  />
                </div>
                <button style={styles.btn} onClick={startCheckout}>Add to Cart</button>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* Cart & Checkout Preview */}
      <section id="cart" style={{ ...styles.section, background: "#fff", color: "#000", textAlign: "center" }}>
        <div style={styles.container}>
          <h2 style={styles.h2}>Your Cart</h2>
          <div style={{ background: "#f3f4f6", borderRadius: 16, padding: 20, maxWidth: 720, margin: "0 auto" }}>
            <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 10 }}>
              <p>SlimBak Towel</p>
              <p>{currency === "USD" ? "$32.99" : "€29.99"}</p>
            </div>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
              <p>Quantity</p>
              <input
                type="number"
                value={qty}
                min={1}
                onChange={(e) => setQty(clampQty(Number(e.target.value)))}
                style={{ ...styles.input, width: 80 }}
              />
            </div>
            <div style={{ display: "flex", justifyContent: "space-between", fontWeight: 700, fontSize: 18, marginBottom: 16 }}>
              <p>Total</p>
              <p>{currency === "USD" ? `$${total.toFixed(2)}` : `€${total.toFixed(2)}`}</p>
            </div>
            <button style={{ ...styles.btn, width: "100%" }} onClick={startCheckout}>Proceed to Checkout</button>
          </div>

          <div style={{ textAlign: "left", maxWidth: 720, margin: "16px auto 0" }}>
            <h3 style={{ fontSize: 18, fontWeight: 700, margin: "12px 0" }}>Currency & Shipping</h3>
            <p style={{ ...styles.p, marginBottom: 6 }}>We ship worldwide from Tunisia & EU warehouses.</p>
            <ul style={{ marginLeft: 18, color: "#4b5563" }}>
              <li>🇪🇺 Europe – prices in € EUR</li>
              <li>🇺🇸 USA – prices in $ USD</li>
              <li>🇹🇳 Tunisia – prices shown as TND (approx), charged in EUR/USD</li>
            </ul>
            <p style={{ ...styles.p, fontSize: 14, color: "#6b7280", marginTop: 8 }}>Currency updates automatically based on your location, or select above.</p>
          </div>
        </div>
      </section>

      {/* AI Upsells */}
      <section style={{ ...styles.section, background: "#0f0f12" }}>
        <div style={styles.container}>
          <h2 style={{ ...styles.h2, textAlign: "center" }}>Recommended For You</h2>
          <p style={{ ...styles.p, textAlign: "center", color: "#9ca3af", marginBottom: 20 }}>
            Personalized by AI based on what customers with similar carts purchased.
          </p>
          <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(260px,1fr))", gap: 16 }}>
            {[{ title: "SlimBak 2‑Pack (Save 15%)", price: "€50.98", was: "€59.98" }, { title: "Magnetic Carry Pouch", price: "€9.99" }, { title: "Extended Warranty (12 mo)", price: "€3.99" }].map((x) => (
              <div key={x.title} style={styles.card}>
                <div style={{ height: 120, background: "#111827", borderRadius: 12, marginBottom: 12 }} />
                <h3 style={{ fontSize: 18, fontWeight: 700, margin: "0 0 6px" }}>{x.title}</h3>
                <div style={{ display: "flex", gap: 10, alignItems: "center", marginBottom: 8 }}>
                  <strong>{x.price}</strong>
                  {x.was ? <span style={{ color: "#9ca3af", textDecoration: "line-through", fontSize: 12 }}>{x.was}</span> : null}
                </div>
                <button style={{ ...styles.btn, width: "100%" }} onClick={() => alert("Added to order (demo)")}>Add To Order</button>
              </div>
            ))}
          </div>
          <div style={{ display: "flex", gap: 12, justifyContent: "center", color: "#9ca3af", marginTop: 18, fontSize: 14, flexWrap: "wrap" }}>
            <span>🔒 Secure Checkout</span>
            <span>🚚 Fast Shipping</span>
            <span>↩️ 30‑Day Returns</span>
            <span>⭐ 4.9/5 Rated</span>
          </div>
        </div>
      </section>

      {/* Testimonials */}
      <section style={styles.section}>
        <div style={{ ...styles.container, textAlign: "center" }}>
          <h2 style={styles.h2}>What Athletes Say</h2>
          <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(260px,1fr))", gap: 16 }}>
            {Array.from({ length: 3 }).map((_, i) => (
              <div key={i} style={styles.card}>
                <div style={{ marginBottom: 8 }}>{"★".repeat(5)}</div>
                <p style={styles.p}>“SlimBak changed my workouts – stylish, practical, and super hygienic.”</p>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* Closing CTA */}
      <section style={{ ...styles.section, background: "#2563eb", textAlign: "center" }}>
        <div style={styles.container}>
          <h2 style={{ ...styles.h2, marginBottom: 12 }}>Elevate your workout. Don’t sweat without SlimBak.</h2>
          <button style={{ ...styles.btnDark }} onClick={startCheckout}>Shop Now</button>
        </div>
      </section>

      {/* Footer with Legal Links */}
      <footer style={styles.footer}>
        <div style={{ marginBottom: 10 }}>
          <a href="#" onClick={(e) => { e.preventDefault(); openPolicy("privacy"); }} style={styles.link}>Privacy Policy</a>
          <a href="#" onClick={(e) => { e.preventDefault(); openPolicy("terms"); }} style={styles.link}>Terms of Service</a>
          <a href="#" onClick={(e) => { e.preventDefault(); openPolicy("refund"); }} style={styles.link}>Refund Policy</a>
          <a href="#" onClick={(e) => { e.preventDefault(); openPolicy("shipping"); }} style={styles.link}>Shipping Policy</a>
        </div>
        <div style={{ fontSize: 13 }}>© {new Date().getFullYear()} SlimBak. All rights reserved.</div>
      </footer>

      {/* Legal Modals */}
      {showPolicy && (
        <div role="dialog" aria-modal="true" style={{ position: "fixed", inset: 0, background: "rgba(0,0,0,.6)", display: "flex", alignItems: "center", justifyContent: "center", padding: 16 }}>
          <div style={{ background: "#0b0b0b", color: "#fff", maxWidth: 800, width: "100%", borderRadius: 16, padding: 20 }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
              <strong style={{ fontSize: 20, textTransform: "capitalize" }}>{showPolicy} policy</strong>
              <button style={styles.btnDark} onClick={closePolicy}>Close</button>
            </div>
            {showPolicy === "privacy" && (
              <p style={styles.p}>SlimBak values your privacy. We only collect personal information necessary to process your order and provide customer service. We do not sell or share your data except with shipping/payment providers. You may request access, updates, or deletion of your data at any time.</p>
            )}
            {showPolicy === "terms" && (
              <ul>
                <li>Products are sold subject to availability.</li>
                <li>Prices are shown in EUR or USD and may include taxes.</li>
                <li>Provide accurate shipping and billing info.</li>
                <li>We may refuse/cancel orders or update prices if needed.</li>
              </ul>
            )}
            {showPolicy === "refund" && (
              <ul>
                <li>Returns accepted within <strong>30 days</strong> if unused and in original packaging.</li>
                <li>Refunds issued to the original method within 5–10 business days.</li>
                <li>Contact: support@slimbak.com</li>
              </ul>
            )}
            {showPolicy === "shipping" && (
              <ul>
                <li>Processing 1–2 business days.</li>
                <li>EU: 3–7 days, USA: 5–10 days, Tunisia: 2–5 days.</li>
                <li>Tracking is provided after dispatch.</li>
              </ul>
            )}
          </div>
        </div>
      )}
    </div>
  );
}

// --------------- Mount for non-Next.js setups ---------------
if (typeof document !== "undefined") {
  const root = document.getElementById("root");
  if (root) createRoot(root).render(<App />);
}

// For Next.js usage, export default the component:
export default App;
