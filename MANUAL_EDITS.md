# Manual edits (fallback if `git apply` rejects the patch)

Open `index.html` in your editor. For each block below, use **Find** to locate the exact text and **Replace** with the new text. Order doesn't matter — each block is independent.

---

## 1. Desktop module session list — replace the whole `SessionList` function body

**Find:**

```jsx
function SessionList({ pack, mod, openSession, setOpenSession, setHoveredObjIdx, typeFilter }) {
  // Group sessions by week, applying the optional type filter
  const filterActive = typeFilter && typeFilter.size > 0;
  const visibleSessions = filterActive
    ? pack.sessions.filter(s => typeFilter.has(s.type))
    : pack.sessions;

  if (filterActive && visibleSessions.length === 0) {
    return (
      <div style={{ padding: "14px 18px", fontSize: 12.5, color: "var(--grey-11)", fontStyle: "italic", borderTop: "1px solid var(--grey-2)", background: "var(--paper)" }}>
        No sessions in this module match the active type filter.
      </div>
    );
  }

  const byWeek = {};
  visibleSessions.forEach(s => { (byWeek[s.week] = byWeek[s.week] || []).push(s); });
  const weeks = Object.keys(byWeek).map(Number).sort((a, b) => a - b);

  return (
    <div style={{ borderTop: "1px solid var(--grey-2)", background: "var(--paper)" }}>
      {weeks.map(w => {
        const wkSessions = byWeek[w];
        return (
          <div key={w}>
            <div style={{ padding: "8px 16px 6px 36px", fontSize: 10.5, letterSpacing: "0.14em", textTransform: "uppercase", color: "var(--grey-7)", fontWeight: 600, background: "var(--grey-1)", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
              <span>Week {w}</span>
            </div>
            {wkSessions.map((s, idx) => {
              const isOpen = openSession === s.id;
              const sessNum = s.n || idx + 1;
              return (
                <div key={s.id} style={{ borderBottom: "1px solid var(--grey-2)" }}>
                  <div
                    style={{ padding: "10px 16px 10px 36px", display: "flex", alignItems: "center", gap: 12, cursor: "pointer", background: isOpen ? "var(--brand-cyan-tint)" : "transparent" }}
                    onClick={(e) => { e.stopPropagation(); setOpenSession(isOpen ? null : s.id); setHoveredObjIdx(null); }}>
                    <Caret open={isOpen} />
                    <div style={{ width: 36, fontSize: 11, color: "var(--grey-11)", flex: "0 0 36px", fontVariantNumeric: "tabular-nums" }}>
                      #{sessNum}
                    </div>
                    <SessionTypePill type={s.type} modId={mod.id} modPhase={mod.phase} />
                    <div style={{ flex: 1, fontWeight: 600, fontSize: 13, color: "var(--session-title)" }}>{s.title}</div>
                    <div style={{ fontSize: 11, color: "var(--grey-11)" }}>{s.duration}min</div>
                    <div style={{ fontSize: 11, color: "var(--grey-11)", marginLeft: 8 }}>
                      <span className="t-num">{s.objectives.length}</span> obj · <span className="t-num">{new Set(s.objectives.flatMap(o => o.mlos)).size}</span> MLOs
                    </div>
                  </div>
                  {isOpen && <SessionObjectives session={s} pack={pack} mod={mod} setHoveredObjIdx={setHoveredObjIdx} />}
                </div>
              );
            })}
          </div>
        );
      })}
    </div>
  );
}
```

**Replace with:**

```jsx
function SessionList({ pack, mod, openSession, setOpenSession, setHoveredObjIdx, typeFilter }) {
  // Sessions are sorted alphabetically by title. Week designation is
  // intentionally not used here — week numbers in source data don't
  // reflect the live academic calendar, so grouping by week produced
  // misleading "schedules". Alphabetical listing is calendar-agnostic.
  const filterActive = typeFilter && typeFilter.size > 0;
  const visibleSessions = filterActive
    ? pack.sessions.filter(s => typeFilter.has(s.type))
    : pack.sessions;

  if (filterActive && visibleSessions.length === 0) {
    return (
      <div style={{ padding: "14px 18px", fontSize: 12.5, color: "var(--grey-11)", fontStyle: "italic", borderTop: "1px solid var(--grey-2)", background: "var(--paper)" }}>
        No sessions in this module match the active type filter.
      </div>
    );
  }

  const sortedSessions = [...visibleSessions].sort((a, b) =>
    (a.title || "").localeCompare(b.title || ""));

  return (
    <div style={{ borderTop: "1px solid var(--grey-2)", background: "var(--paper)" }}>
      {sortedSessions.map((s, idx) => {
        const isOpen = openSession === s.id;
        const sessNum = s.n || idx + 1;
        return (
          <div key={s.id} style={{ borderBottom: "1px solid var(--grey-2)" }}>
            <div
              style={{ padding: "10px 16px 10px 36px", display: "flex", alignItems: "center", gap: 12, cursor: "pointer", background: isOpen ? "var(--brand-cyan-tint)" : "transparent" }}
              onClick={(e) => { e.stopPropagation(); setOpenSession(isOpen ? null : s.id); setHoveredObjIdx(null); }}>
              <Caret open={isOpen} />
              <div style={{ width: 36, fontSize: 11, color: "var(--grey-11)", flex: "0 0 36px", fontVariantNumeric: "tabular-nums" }}>
                #{sessNum}
              </div>
              <SessionTypePill type={s.type} modId={mod.id} modPhase={mod.phase} />
              <div style={{ flex: 1, fontWeight: 600, fontSize: 13, color: "var(--session-title)" }}>{s.title}</div>
              <div style={{ fontSize: 11, color: "var(--grey-11)" }}>{s.duration}min</div>
              <div style={{ fontSize: 11, color: "var(--grey-11)", marginLeft: 8 }}>
                <span className="t-num">{s.objectives.length}</span> obj · <span className="t-num">{new Set(s.objectives.flatMap(o => o.mlos)).size}</span> MLOs
              </div>
            </div>
            {isOpen && <SessionObjectives session={s} pack={pack} mod={mod} setHoveredObjIdx={setHoveredObjIdx} />}
          </div>
        );
      })}
    </div>
  );
}
```

---

## 2. Mobile module session list — sort A–Z, drop the left "Week N" column

**Find:**

```jsx
      {sessions.length > 0 ? (
        <InfoBlock label={"Sessions · " + sessions.length}>
          <div style={{ display: "flex", flexDirection: "column", paddingTop: 4 }}>
            {sessions.map((s, i) => (
              <button key={s.id || i} onClick={() => openSession(s)}
                style={{ display: "flex", alignItems: "flex-start", gap: 10, padding: "11px 0", border: "none", background: "transparent", borderTop: i === 0 ? "none" : "1px solid " + BRAND.grey2, textAlign: "left", cursor: "pointer", width: "100%", fontFamily: "inherit", WebkitTapHighlightColor: "rgba(0,0,0,0.06)" }}>
                <div style={{ flex: "0 0 52px", paddingTop: 2 }}>
                  <div style={{ fontSize: 10, color: BRAND.grey11, textTransform: "uppercase", letterSpacing: 0.4, fontWeight: 700 }}>{s.week != null ? "Week " + s.week : "—"}</div>
                </div>
                <div style={{ flex: 1, minWidth: 0 }}>
                  <div style={{ fontSize: 13.5, fontWeight: 600, color: BRAND.sessionTitle || "#0B3B7A", lineHeight: 1.35 }}>{s.title}</div>
                  <div style={{ fontSize: 11.5, color: BRAND.grey11, marginTop: 3, fontWeight: 500 }}>
                    {[moduleTypeCode(s), s.duration ? s.duration + " min" : null].filter((x) => x && x !== "Other").join(" · ") || "—"}
                  </div>
                </div>
                <IconChevR size={14} stroke={BRAND.grey3} sw={2} style={{ marginTop: 4, flexShrink: 0 }}/>
              </button>
            ))}
          </div>
        </InfoBlock>
      ) : (
```

**Replace with:**

```jsx
      {sessions.length > 0 ? (
        <InfoBlock label={"Sessions · " + sessions.length}>
          <div style={{ display: "flex", flexDirection: "column", paddingTop: 4 }}>
            {[...sessions].sort((a, b) => (a.title || "").localeCompare(b.title || "")).map((s, i) => (
              <button key={s.id || i} onClick={() => openSession(s)}
                style={{ display: "flex", alignItems: "flex-start", gap: 10, padding: "11px 0", border: "none", background: "transparent", borderTop: i === 0 ? "none" : "1px solid " + BRAND.grey2, textAlign: "left", cursor: "pointer", width: "100%", fontFamily: "inherit", WebkitTapHighlightColor: "rgba(0,0,0,0.06)" }}>
                <div style={{ flex: 1, minWidth: 0 }}>
                  <div style={{ fontSize: 13.5, fontWeight: 600, color: BRAND.sessionTitle || "#0B3B7A", lineHeight: 1.35 }}>{s.title}</div>
                  <div style={{ fontSize: 11.5, color: BRAND.grey11, marginTop: 3, fontWeight: 500 }}>
                    {[moduleTypeCode(s), s.duration ? s.duration + " min" : null].filter((x) => x && x !== "Other").join(" · ") || "—"}
                  </div>
                </div>
                <IconChevR size={14} stroke={BRAND.grey3} sw={2} style={{ marginTop: 4, flexShrink: 0 }}/>
              </button>
            ))}
          </div>
        </InfoBlock>
      ) : (
```

---

## 3. Strip "Week N" / "Wk N" from subtitles and list eyebrows

Each block below is a single-line find/replace. Some occur more than once — replace **every** occurrence.

### 3a. Search index builder

**Find:**

```js
        sub: `${modShort}${s.week ? " · Week " + s.week : ""}${s.type ? " · " + s.type : ""}`,
```

**Replace with:**

```js
        sub: `${modShort}${s.type ? " · " + s.type : ""}`,
```

### 3b. Desktop `openSession` (ModuleDetailBody)

**Find:**

```js
      sub: moduleShort + (s.week ? " · Week " + s.week : "") + (s.type ? " · " + s.type : ""),
```

**Replace with:**

```js
      sub: moduleShort + (s.type ? " · " + s.type : ""),
```

### 3c. `openRelated` (SessionDetailBody)

**Find:**

```js
      sub: (rmod.short || rmod.id) + (rs.week ? " · Week " + rs.week : "") + (rs.type ? " · " + rs.type : ""),
```

**Replace with:**

```js
      sub: (rmod.short || rmod.id) + (rs.type ? " · " + rs.type : ""),
```

### 3d. Browse-tree leaf tap (`tapNode`)

**Find:**

```js
        sub: moduleShort + (sp.session.week ? " · Week " + sp.session.week : "") + (sp.session.type ? " · " + sp.session.type : ""),
```

**Replace with:**

```js
        sub: moduleShort + (sp.session.type ? " · " + sp.session.type : ""),
```

### 3e. MEPO drill-down `sItem.sub` (occurs twice — replace both)

**Find:**

```js
                sub: `${r.mod.short || r.mod.id}${r.session.week ? " · Week " + r.session.week : ""}${r.session.type ? " · " + r.session.type : ""}`,
```

**Replace with:**

```js
                sub: `${r.mod.short || r.mod.id}${r.session.type ? " · " + r.session.type : ""}`,
```

### 3f. Mobile MEPO/AOC list eyebrow (occurs twice — replace both)

**Find:**

```jsx
                  {r.mod.short || r.mod.name}{r.session.week ? " · Wk " + r.session.week : ""}
```

**Replace with:**

```jsx
                  {r.mod.short || r.mod.name}
```

### 3g. Session side panel sub-line

**Find:**

```jsx
        {mod?.name}{session.week ? " · Week " + session.week : ""} · {window.shortTypeCode(session.type, mod?.id, mod?.phase)}
```

**Replace with:**

```jsx
        {mod?.name} · {window.shortTypeCode(session.type, mod?.id, mod?.phase)}
```

---

## Done

Save, commit, push:

```bash
git checkout -b sessions-alphabetical
git add index.html
git commit -m "Sort module sessions alphabetically; remove week designation from list views"
git push -u origin sessions-alphabetical
```
