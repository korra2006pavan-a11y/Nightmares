import Link from "next/link";
import { and, desc, eq } from "drizzle-orm";
import { db } from "@/db";
import { users } from "@/db/schema";
import { getSessionUser } from "@/lib/auth";
import { ensureSeeded } from "@/lib/seed";
import { ADMIN_UPI_ID, FEATURES, PLAN, SITE } from "@/lib/config";
import { CITIES, formatDistance, haversineKm } from "@/lib/geo";

export const dynamic = "force-dynamic";

const STEPS = [
  {
    n: "01",
    title: "Create your account",
    body: "Login ID, password, photo and a short intro. Email + mobile OTP are sent instantly.",
  },
  {
    n: "02",
    title: "Get verified",
    body: "Upload a live selfie. Trust & Safety match it with your profile photo before your badge goes live.",
  },
  {
    n: "03",
    title: "Pay ₹99 & upload proof",
    body: `Pay to ${ADMIN_UPI_ID}, upload the screenshot with the UTR. Our checker validates it in seconds.`,
  },
  {
    n: "04",
    title: "Meet nearby",
    body: "Unlock photos, choose who sees your number, chat, share pictures and plan the first meet-up.",
  },
];

const FAQ = [
  {
    q: "Why is there a ₹99 monthly fee?",
    a: "A small paid gate is the cheapest way to keep time-wasters, catfish and bulk fake profiles out. ₹99 covers verification review, moderation and hosting for 30 days. There is no auto-debit — you simply pay again when the month ends.",
  },
  {
    q: "How does the UTR checker work?",
    a: "After you pay to the admin UPI ID, you upload the payment screenshot and type the 12-digit UTR/RRN. The checker validates the format, confirms the reference has never been used on Milap, checks the amount and velocity, then activates 30 days of access. Suspicious references are parked for a human admin review.",
  },
  {
    q: "Who can see my number and my photos?",
    a: "Only verified, paid members can see profile photos. Your mobile number stays hidden by default and is shared only after a mutual like when you switch on 'Show my number' in Privacy. You can switch it off again any time.",
  },
  {
    q: "How are fake profiles removed?",
    a: "Every profile carries a trust score built from OTP verification, selfie review, profile completeness and member reports. Three open reports quarantine an account automatically; admins can verify, flag, ban or permanently delete an account from the moderation console.",
  },
  {
    q: "Is my location shared exactly?",
    a: "No. We use your coordinates to rank people by distance and only ever show an approximate distance like '1.4 km away'. Your exact coordinates are never displayed to another member.",
  },
];

export default async function LandingPage() {
  await ensureSeeded();
  const viewer = await getSessionUser();

  const featured = await db
    .select()
    .from(users)
    .where(and(eq(users.role, "member"), eq(users.status, "active")))
    .orderBy(desc(users.trustScore))
    .limit(4);

  const origin = CITIES[0];

  return (
    <div className="relative overflow-hidden">
      {/* Nav */}
      <header className="sticky top-0 z-40 border-b border-white/10 bg-ink/70 backdrop-blur-xl">
        <div className="mx-auto flex max-w-6xl items-center justify-between px-5 py-4">
          <Link href="/" className="flex items-center gap-2.5">
            <span className="grid h-9 w-9 place-items-center rounded-2xl bg-gradient-to-br from-rose to-rose-deep text-lg shadow-lg shadow-rose/30">
              ❤
            </span>
            <span className="display text-xl font-semibold tracking-tight">{SITE.name}</span>
            <span className="hidden rounded-full border border-mint/30 bg-mint/10 px-2 py-0.5 text-[10px] font-bold uppercase tracking-wider text-mint sm:inline">
              verified
            </span>
          </Link>
          <nav className="hidden items-center gap-7 text-sm text-cream/70 md:flex">
            <a href="#how" className="hover:text-cream">How it works</a>
            <a href="#safety" className="hover:text-cream">Safety</a>
            <a href="#pricing" className="hover:text-cream">Pricing</a>
            <a href="#faq" className="hover:text-cream">FAQ</a>
          </nav>
          <div className="flex items-center gap-2">
            <Link href="/admin/login" className="hidden text-xs text-cream/45 hover:text-cream sm:block">
              Admin
            </Link>
            {viewer ? (
              <Link href={viewer.role === "admin" ? "/admin" : "/app"} className="btn-primary btn-sm">
                Open app
              </Link>
            ) : (
              <>
                <Link href="/login" className="btn-ghost btn-sm">Sign in</Link>
                <Link href="/signup" className="btn-primary btn-sm">Join · ₹{PLAN.amount}</Link>
              </>
            )}
          </div>
        </div>
      </header>

      {/* Hero */}
      <section className="relative mx-auto grid max-w-6xl gap-14 px-5 pb-20 pt-14 lg:grid-cols-[1.05fr_0.95fr] lg:pt-20">
        <div className="fade-up">
          <span className="chip border-rose/30 bg-rose/10 text-rose">
            <span className="live-dot h-1.5 w-1.5 rounded-full bg-rose" />
            Andhra Pradesh · Telangana · live now
          </span>
          <h1 className="display mt-6 text-5xl leading-[1.02] sm:text-6xl">
            Find your person,
            <br />
            <span className="bg-gradient-to-r from-rose via-gold to-cream bg-clip-text text-transparent">
              a few streets away.
            </span>
          </h1>
          <p className="mt-6 max-w-xl text-[15px] leading-relaxed text-cream/65">
            Milap connects verified single adults for real relationships and honest meet-ups.
            Photo + selfie checks, email and mobile OTP, nearest-first discovery, private contact
            sharing and a moderated members-only pool — all for ₹{PLAN.amount} a month.
          </p>

          <div className="mt-8 flex flex-wrap items-center gap-3">
            <Link href="/signup" className="btn-primary">
              Create verified profile
              <span aria-hidden>→</span>
            </Link>
            <Link href="/login" className="btn-ghost">
              I already have a login
            </Link>
          </div>

          <div className="mt-8 flex flex-wrap gap-2">
            {["Email + mobile OTP", "Selfie matched photo", "UTR-checked payments", "Contact stays private"].map(
              (item) => (
                <span key={item} className="chip">
                  <span className="text-mint">✓</span> {item}
                </span>
              ),
            )}
          </div>

          <dl className="mt-10 grid max-w-lg grid-cols-3 gap-4 text-center">
            {[
              ["₹99", "per month, no auto-debit"],
              ["OTP", "email + mobile check"],
              ["24h", "fake profile turnover"],
            ].map(([value, caption]) => (
              <div key={value} className="card px-3 py-4">
                <dt className="display text-2xl text-cream">{value}</dt>
                <dd className="mt-1 text-[11px] leading-tight text-cream/50">{caption}</dd>
              </div>
            ))}
          </dl>
        </div>

        {/* Featured people card stack */}
        <div className="fade-up relative">
          <div className="card relative overflow-hidden p-4">
            <div className="mb-3 flex items-center justify-between">
              <div>
                <p className="text-[11px] uppercase tracking-[0.2em] text-cream/40">Nearby right now</p>
                <p className="display text-lg">{origin.name} · within 60 km</p>
              </div>
              <span className="badge-verified">
                <span className="live-dot h-1.5 w-1.5 rounded-full bg-mint" /> live
              </span>
            </div>
            <div className="grid grid-cols-2 gap-3">
              {featured.map((person) => {
                const km = haversineKm(origin.lat, origin.lng, person.lat ?? origin.lat, person.lng ?? origin.lng);
                return (
                  <div key={person.id} className="group relative overflow-hidden rounded-2xl border border-white/10">
                    <div className="aspect-[4/5] w-full bg-ink-soft">
                      {(person.photoMediaId || person.photoUrl) && (
                        // eslint-disable-next-line @next/next/no-img-element
                        <img
                          src={person.photoMediaId ? `/api/media/${person.photoMediaId}` : person.photoUrl!}
                          alt={person.fullName}
                          className="h-full w-full object-cover transition duration-500 group-hover:scale-105"
                        />
                      )}
                    </div>
                    <div className="absolute inset-x-0 bottom-0 bg-gradient-to-t from-black/90 via-black/40 to-transparent p-2.5">
                      <div className="flex items-center gap-1.5 text-[13px] font-semibold">
                        {person.fullName.split(" ")[0]}, {new Date().getFullYear() - person.birthYear}
                        {person.identityVerified && <span className="text-mint">✓</span>}
                      </div>
                      <div className="text-[10px] text-cream/60">
                        {formatDistance(km)} · {person.area || person.city}
                      </div>
                    </div>
                  </div>
                );
              })}
            </div>
            <div className="mt-4 flex items-center gap-3 rounded-2xl border border-white/10 bg-black/30 p-3">
              <div className="grid h-10 w-10 place-items-center rounded-xl bg-rose/15 text-lg">🔒</div>
              <p className="text-[11px] leading-relaxed text-cream/55">
                Photos, numbers and chat unlock only for members whose ₹{PLAN.amount} UTR payment is
                verified.
              </p>
            </div>
          </div>
          <div className="absolute -bottom-6 -left-4 hidden rounded-2xl border border-mint/30 bg-ink-card/95 px-4 py-3 shadow-xl sm:block">
            <p className="text-[10px] uppercase tracking-widest text-cream/40">Trust score</p>
            <p className="display text-xl text-mint">100 / 100 · low risk</p>
          </div>
        </div>
      </section>

      <div className="divider" />

      {/* How it works */}
      <section id="how" className="mx-auto max-w-6xl px-5 py-20">
        <div className="max-w-2xl">
          <p className="text-[11px] uppercase tracking-[0.24em] text-rose">Four steps, no games</p>
          <h2 className="display mt-3 text-4xl sm:text-[42px]">
            From sign-up to a verified meet-up
          </h2>
          <p className="mt-4 text-sm leading-relaxed text-cream/60">
            Every account passes identity checks before it appears in anyone&apos;s discovery feed.
            Membership access starts only after your payment UTR clears the checker.
          </p>
        </div>
        <div className="mt-10 grid gap-4 md:grid-cols-2 lg:grid-cols-4">
          {STEPS.map((step) => (
            <div key={step.n} className="card p-6">
              <span className="display text-3xl text-rose/70">{step.n}</span>
              <h3 className="mt-4 text-base font-semibold">{step.title}</h3>
              <p className="mt-2 text-[13px] leading-relaxed text-cream/55">{step.body}</p>
            </div>
          ))}
        </div>
      </section>

      {/* Features */}
      <section className="mx-auto max-w-6xl px-5 pb-20">
        <div className="grid gap-4 md:grid-cols-2">
          {FEATURES.map((feature) => (
            <div key={feature.title} className="card flex gap-4 p-6">
              <span className="grid h-11 w-11 shrink-0 place-items-center rounded-2xl bg-white/[0.06] text-xl">
                {feature.icon}
              </span>
              <div>
                <h3 className="text-base font-semibold">{feature.title}</h3>
                <p className="mt-1.5 text-[13px] leading-relaxed text-cream/55">{feature.body}</p>
              </div>
            </div>
          ))}
        </div>
      </section>

      {/* Safety */}
      <section id="safety" className="border-y border-white/10 bg-black/20">
        <div className="mx-auto grid max-w-6xl gap-10 px-5 py-20 lg:grid-cols-2">
          <div>
            <p className="text-[11px] uppercase tracking-[0.24em] text-mint">Fake-profile defence</p>
            <h2 className="display mt-3 text-4xl sm:text-[42px]">
              Bots, catfish and time-wasters get removed.
            </h2>
            <p className="mt-4 text-sm leading-relaxed text-cream/60">
              Milap runs a layered verification stack. Every signal adds to a trust score; member
              reports subtract from it. Anything below the threshold never reaches discovery, and
              admins can permanently delete an account.
            </p>
            <ul className="mt-6 space-y-3 text-[13px] text-cream/70">
              {[
                "6-digit OTP on email and mobile — no disposable sign-ups",
                "Live selfie compared with the profile photo by a human reviewer",
                "One UTR can activate exactly one account — screenshot reuse is auto-blocked",
                "Repeating-digit / high-velocity UTRs parked for manual approval",
                "3 open member reports = automatic quarantine",
                "Admin console: verify, flag, ban, downgrade or delete",
              ].map((line) => (
                <li key={line} className="flex gap-3">
                  <span className="mt-0.5 text-mint">✓</span>
                  <span>{line}</span>
                </li>
              ))}
            </ul>
          </div>
          <div className="card-solid p-2">
            <div className="rounded-[20px] bg-gradient-to-br from-ink-soft to-black/60 p-6">
              <div className="flex items-center justify-between">
                <p className="text-xs uppercase tracking-widest text-cream/40">Moderator view</p>
                <span className="badge-danger">under review</span>
              </div>
              <div className="mt-5 flex gap-4">
                <div className="grid h-20 w-20 shrink-0 place-items-center rounded-2xl border border-white/10 bg-white/5 text-2xl">
                  🕵️
                </div>
                <div className="text-[13px]">
                  <p className="font-semibold">Suspicious profile #IMRAN</p>
                  <p className="text-cream/50">No selfie · No photo · 3 reports open</p>
                  <p className="mt-2 text-rose">Trust score 12 / 100 — quarantined automatically</p>
                </div>
              </div>
              <div className="mt-5 space-y-2 text-[12px]">
                {[
                  ["Email OTP", "verified", true],
                  ["Mobile OTP", "not verified", false],
                  ["Selfie match", "missing", false],
                  ["UTR payment", "not submitted", false],
                ].map(([label, value, good]) => (
                  <div key={String(label)} className="flex items-center justify-between rounded-xl border border-white/10 bg-black/20 px-3 py-2">
                    <span className="text-cream/60">{label}</span>
                    <span className={good ? "text-mint" : "text-rose"}>{value}</span>
                  </div>
                ))}
              </div>
              <div className="mt-5 flex gap-2">
                <span className="btn-mint btn-sm flex-1">Verify</span>
                <span className="btn-ghost btn-sm flex-1">Remove</span>
              </div>
              <p className="mt-4 text-[11px] text-cream/35">
                Admin console screenshot · real controls live behind admin login
              </p>
            </div>
          </div>
        </div>
      </section>

      {/* Pricing */}
      <section id="pricing" className="mx-auto max-w-6xl px-5 py-20">
        <div className="mx-auto max-w-xl text-center">
          <p className="text-[11px] uppercase tracking-[0.24em] text-gold">Simple, honest pricing</p>
          <h2 className="display mt-3 text-4xl sm:text-[42px]">One plan. ₹{PLAN.amount} a month.</h2>
          <p className="mt-4 text-sm text-cream/60">
            No hidden coins, no boosts to buy. Membership gives full access to verified profiles,
            photo viewing, contact sharing and unlimited chat.
          </p>
        </div>
        <div className="mx-auto mt-10 grid max-w-4xl gap-4 md:grid-cols-[1.1fr_0.9fr]">
          <div className="card relative overflow-hidden p-7">
            <div className="absolute right-5 top-5 badge-verified">30 days</div>
            <h3 className="display text-2xl">{PLAN.label}</h3>
            <p className="mt-1 text-xs text-cream/45">Activated after UTR verification</p>
            <p className="mt-5 flex items-end gap-2">
              <span className="display text-5xl">₹{PLAN.amount}</span>
              <span className="pb-1 text-xs text-cream/45">/ month</span>
            </p>
            <ul className="mt-6 space-y-2.5 text-[13px] text-cream/70">
              {[
                "Nearest-first verified discovery",
                "Unlimited likes with mutual-match chat",
                "Photo sharing in chat (toggleable)",
                "Contact reveal only on mutual like",
                "Admin-verified identity badge",
                "Fake-profile reporting & fast removal",
              ].map((item) => (
                <li key={item} className="flex gap-2.5">
                  <span className="text-mint">✓</span> {item}
                </li>
              ))}
            </ul>
            <Link href="/signup" className="btn-primary mt-7 w-full">
              Join Milap · pay ₹{PLAN.amount}
            </Link>
          </div>
          <div className="card p-7">
            <p className="text-[11px] uppercase tracking-[0.2em] text-cream/40">How to pay</p>
            <h3 className="display mt-2 text-xl">UPI only, verified by UTR</h3>
            <div className="mt-4 rounded-2xl border border-gold/25 bg-gold/[0.07] p-4">
              <p className="text-[10px] uppercase tracking-widest text-gold/70">Admin UPI ID</p>
              <p className="mt-1 break-all font-mono text-base text-gold">{ADMIN_UPI_ID}</p>
              <p className="mt-1 text-[11px] text-cream/45">Payee: {SITE.name} Membership</p>
            </div>
            <ol className="mt-5 space-y-3 text-[13px] text-cream/65">
              <li><span className="text-rose">1.</span> Pay ₹{PLAN.amount} from any UPI app to the ID above.</li>
              <li><span className="text-rose">2.</span> Screenshot the success page.</li>
              <li><span className="text-rose">3.</span> Upload it inside Milap with the 12-digit UTR.</li>
              <li><span className="text-rose">4.</span> Checker validates and unlocks access instantly.</li>
            </ol>
            <p className="mt-5 rounded-xl border border-white/10 bg-black/25 p-3 text-[11px] leading-relaxed text-cream/45">
              Each UTR can activate only one account. Reused or fabricated references are blocked,
              and repeated attempts send the account for manual review.
            </p>
          </div>
        </div>
      </section>

      {/* FAQ */}
      <section id="faq" className="border-t border-white/10 bg-black/20">
        <div className="mx-auto max-w-3xl px-5 py-20">
          <h2 className="display text-4xl">Questions, answered</h2>
          <div className="mt-8 space-y-3">
            {FAQ.map((item) => (
              <details key={item.q} className="card group p-5 [&_summary::-webkit-details-marker]:hidden">
                <summary className="flex cursor-pointer items-center justify-between gap-4 text-[15px] font-medium">
                  {item.q}
                  <span className="text-rose transition group-open:rotate-45">+</span>
                </summary>
                <p className="mt-3 text-[13px] leading-relaxed text-cream/60">{item.a}</p>
              </details>
            ))}
          </div>
          <div className="mt-10 card flex flex-col items-center gap-4 p-8 text-center">
            <h3 className="display text-2xl">Ready to meet someone real?</h3>
            <p className="max-w-md text-[13px] text-cream/55">
              Verification takes under five minutes. Membership starts at ₹{PLAN.amount} with no
              auto-debit.
            </p>
            <div className="flex flex-wrap justify-center gap-3">
              <Link href="/signup" className="btn-primary">Create my profile</Link>
              <Link href="/login" className="btn-ghost">Sign in</Link>
            </div>
          </div>
        </div>
      </section>

      <footer className="border-t border-white/10">
        <div className="mx-auto flex max-w-6xl flex-col gap-4 px-5 py-8 text-[12px] text-cream/40 sm:flex-row sm:items-center sm:justify-between">
          <p>© {new Date().getFullYear()} {SITE.name}. For adults 18+. Meet in public places.</p>
          <div className="flex flex-wrap items-center gap-5">
            <a href="#safety" className="hover:text-cream">Safety</a>
            <a href="#pricing" className="hover:text-cream">Pricing</a>
            <Link href="/admin/login" className="hover:text-cream">Admin console</Link>
            <a href={`mailto:${SITE.supportEmail}`} className="hover:text-cream">Support</a>
          </div>
        </div>
      </footer>
    </div>
  );
}
