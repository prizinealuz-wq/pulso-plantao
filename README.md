<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Pulso · Plantão</title>
<link rel="icon" href="data:,">
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;600;700&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@500;600;700&display=swap');
  * { box-sizing: border-box; }
  html, body { margin: 0; padding: 0; background: #10141B; }
  body { font-family: 'Inter', sans-serif; color: #EDEFF3; }
  #root { min-height: 100vh; }
  button, input, select { font: inherit; }
  input[type=number]::-webkit-inner-spin-button { opacity: 1; }
</style>
</head>
<body>
<div id="root"></div>

<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

<script type="text/babel">
const { useState, useEffect } = React;

const VARS = {
  bg: "#10141B",
  surface: "#1A2028",
  surface2: "#222A34",
  border: "#2C3543",
  borderSoft: "#242C38",
  text: "#EDEFF3",
  textMuted: "#8892A1",
  textFaint: "#5C6675",
  teal: "#4FD1C5",
  tealSoft: "rgba(79, 209, 197, 0.14)",
  amber: "#F2A65A",
  amberSoft: "rgba(242, 166, 90, 0.16)",
  red: "#E5707A",
};

function formatBRL(v) {
  return (v || 0).toLocaleString("pt-BR", { style: "currency", currency: "BRL" });
}

function iconBtnStyle(active) {
  return {
    width: 32,
    height: 32,
    borderRadius: 8,
    border: `1px solid ${active ? VARS.teal : VARS.border}`,
    background: active ? VARS.tealSoft : VARS.surface2,
    color: active ? VARS.teal : "#AEB6C2",
    display: "flex",
    alignItems: "center",
    justifyContent: "center",
    cursor: "pointer",
    fontSize: 14,
  };
}

function ConfigField({ label, value, onChange }) {
  return (
    <label style={{ display: "block" }}>
      <span style={{
        display: "block", fontSize: 10.5, color: VARS.textFaint,
        marginBottom: 4, textTransform: "uppercase", letterSpacing: "0.04em",
      }}>{label}</span>
      <input
        type="number"
        value={value}
        onChange={(e) => onChange(e.target.value)}
        style={{
          width: "100%", background: VARS.surface2, border: `1px solid ${VARS.border}`,
          borderRadius: 8, padding: "7px 9px", color: VARS.text,
          fontFamily: "'JetBrains Mono', monospace", fontSize: 13,
          outline: "none", boxSizing: "border-box",
        }}
      />
    </label>
  );
}

function HourCard({ hour, config, value, onSet, onRemove, canRemove }) {
  const squares = Array.from({ length: config.maxPatients }, (_, i) => i + 1);
  const isExtra = hour.count > config.included;

  return (
    <div style={{
      background: VARS.surface, border: `1px solid ${VARS.border}`,
      borderRadius: 14, padding: 14, marginBottom: 12,
    }}>
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
        <span style={{ fontFamily: "'Space Grotesk', sans-serif", fontWeight: 600, fontSize: 14.5 }}>
          {hour.label}
        </span>
        <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
          <span style={{
            fontFamily: "'JetBrains Mono', monospace", fontWeight: 700, fontSize: 16,
            color: isExtra ? VARS.amber : hour.count > 0 ? VARS.teal : VARS.textFaint,
          }}>
            {formatBRL(value)}
          </span>
          {canRemove && (
            <button onClick={onRemove} title="Remover hora" style={{ ...iconBtnStyle(), color: VARS.red }}>
              ✕
            </button>
          )}
        </div>
      </div>

      <div style={{ display: "flex", gap: 5 }}>
        {squares.map((n) => {
          const checked = n <= hour.count;
          const extraZone = n > config.included;
          return (
            <button
              key={n}
              onClick={() => onSet(n === hour.count ? n - 1 : n)}
              style={{
                flex: 1, aspectRatio: "1 / 1", minWidth: 0, borderRadius: 7,
                border: `1.5px solid ${checked ? (extraZone ? VARS.amber : VARS.teal) : (extraZone ? VARS.borderSoft : VARS.border)}`,
                background: checked ? (extraZone ? VARS.amberSoft : VARS.tealSoft) : "transparent",
                color: checked ? (extraZone ? VARS.amber : VARS.teal) : VARS.textFaint,
                fontFamily: "'JetBrains Mono', monospace", fontSize: 11.5, fontWeight: 600,
                cursor: "pointer", transition: "all 0.12s ease", position: "relative",
              }}
            >
              {n}
              {n === config.included && (
                <span style={{
                  position: "absolute", right: -3.5, top: "50%", transform: "translateY(-50%)",
                  width: 1, height: "70%", background: VARS.textFaint, opacity: 0.5,
                }} />
              )}
            </button>
          );
        })}
      </div>

      <div style={{ marginTop: 8, fontSize: 11, color: VARS.textFaint }}>
        {hour.count === 0
          ? "Nenhum paciente registrado"
          : isExtra
          ? `${config.included} × ${formatBRL(config.baseValue)} + ${Math.min(hour.count, config.maxPatients) - config.included} extra × ${formatBRL(config.extraValue)}`
          : `${hour.count} × ${formatBRL(config.baseValue)} (base)`}
      </div>
    </div>
  );
}

function CalendarView({ history, selectedYear, selectedMonth, onSelectDay }) {
  const daysInMonth = new Date(selectedYear, selectedMonth + 1, 0).getDate();
  const firstDayIndex = new Date(selectedYear, selectedMonth, 1).getDay();
  
  const days = [];
  for (let i = 0; i < firstDayIndex; i++) days.push(null);
  for (let d = 1; d <= daysInMonth; d++) days.push(d);

  const getShiftForDay = (d) => {
    if (!d) return [];
    const dateStr = `${selectedYear}-${String(selectedMonth + 1).padStart(2, '0')}-${String(d).padStart(2, '0')}`;
    return history.filter(p => p.date === dateStr);
  };

  return (
    <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 14, padding: 14, marginBottom: 16 }}>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(7, 1fr)", gap: 4, textAlign: "center", fontSize: 11, color: VARS.textFaint, marginBottom: 8, fontWeight: 600 }}>
        <span>DOM</span><span>SEG</span><span>TER</span><span>QUA</span><span>QUI</span><span>SEX</span><span>SÁB</span>
      </div>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(7, 1fr)", gap: 4 }}>
        {days.map((day, idx) => {
          if (!day) return <div key={idx} style={{ height: 44 }} />;
          
          const dateStr = `${selectedYear}-${String(selectedMonth + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
          const shifts = getShiftForDay(day);
          const hasShift = shifts.length > 0;
          const totalDayValue = hasShift ? shifts.reduce((s, p) => s + p.totalValue, 0) : 0;

          return (
            <div
              key={idx}
              onClick={() => onSelectDay(day, dateStr, shifts)}
              style={{
                height: 44,
                borderRadius: 8,
                border: `1px solid ${hasShift ? VARS.teal : VARS.borderSoft}`,
                background: hasShift ? VARS.tealSoft : VARS.surface2,
                display: "flex",
                flexDirection: "column",
                alignItems: "center",
                justifyContent: "center",
                cursor: "pointer",
                position: "relative",
              }}
            >
              <span style={{ fontSize: 12, fontWeight: hasShift ? "700" : "400", color: hasShift ? VARS.teal : VARS.textMuted }}>
                {day}
              </span>
              {hasShift && (
                <span style={{ fontSize: 8.5, fontFamily: "'JetBrains Mono', monospace", color: VARS.teal, fontWeight: 600 }}>
                  R${Math.round(totalDayValue)}
                </span>
              )}
            </div>
          );
        })}
      </div>
    </div>
  );
}

function PlantaoTracker() {
  const [activeTab, setActiveTab] = useState("tracker");
  const [config, setConfig] = useState(() => {
    const saved = localStorage.getItem("pulso_config");
    return saved ? JSON.parse(saved) : { baseValue: 25, included: 4, extraValue: 7, maxPatients: 10 };
  });
  
  const [showConfig, setShowConfig] = useState(false);
  const [hours, setHours] = useState([{ id: 1, label: "Hora 1", count: 0 }]);
  const [nextId, setNextId] = useState(2);
  
  const [shiftDate, setShiftDate] = useState(() => new Date().toISOString().split('T')[0]);
  const [shiftLocation, setShiftLocation] = useState("Hospital Principal");
  const [shiftType, setShiftType] = useState("Noturno (12h)");

  const [history, setHistory] = useState(() => {
    const saved = localStorage.getItem("pulso_history");
    return saved ? JSON.parse(saved) : [];
  });

  const [viewDate, setViewDate] = useState(new Date());
  const [selectedDayModal, setSelectedDayModal] = useState(null);
  const [editMode, setEditMode] = useState("quick");
  const [expandedShiftId, setExpandedShiftId] = useState(null);

  useEffect(() => {
    localStorage.setItem("pulso_config", JSON.stringify(config));
  }, [config]);

  useEffect(() => {
    localStorage.setItem("pulso_history", JSON.stringify(history));
  }, [history]);

  const hourValue = (count, cfg = config) => {
    if (count <= 0) return 0;
    const capped = Math.min(count, cfg.maxPatients);
    const baseCount = Math.min(capped, cfg.included);
    const extraCount = Math.max(0, capped - cfg.included);
    return baseCount * cfg.baseValue + extraCount * cfg.extraValue;
  };

  const setCount = (id, val) => {
    setHours((hs) => hs.map((h) => h.id === id ? { ...h, count: Math.max(0, Math.min(config.maxPatients, val)) } : h));
  };

  const addHour = () => {
    setHours((hs) => [...hs, { id: nextId, label: `Hora ${hs.length + 1}`, count: 0 }]);
    setNextId((n) => n + 1);
  };

  const removeHour = (id) => {
    setHours((hs) => {
      const filtered = hs.filter((h) => h.id !== id);
      return filtered.map((h, i) => ({ ...h, label: `Hora ${i + 1}` }));
    });
  };

  const resetAll = () => {
    if (window.confirm("Zerar contadores do plantão atual?")) {
      setHours([{ id: nextId, label: "Hora 1", count: 0 }]);
      setNextId((n) => n + 1);
    }
  };

  const totalPatients = hours.reduce((s, h) => s + h.count, 0);
  const totalValue = hours.reduce((s, h) => s + hourValue(h.count), 0);
  const avgPerHour = hours.length > 0 ? totalValue / hours.length : 0;

  const saveShift = () => {
    if (totalPatients === 0) {
      alert("Registre ao menos 1 paciente antes de salvar o plantão.");
      return;
    }

    const newShift = {
      id: Date.now(),
      date: shiftDate,
      location: shiftLocation,
      type: shiftType,
      totalPatients,
      totalValue,
      hoursCount: hours.length,
      hoursData: hours,
      config: { ...config }
    };

    setHistory([newShift, ...history.filter(h => h.date !== shiftDate)]);
    alert("Plantão salvo no histórico com sucesso!");
    setHours([{ id: nextId, label: "Hora 1", count: 0 }]);
    setNextId((n) => n + 1);
    setActiveTab("history");
  };

  const saveQuickShift = (dateStr, location, type, totalPacs, durationHours) => {
    const pacs = Number(totalPacs) || 0;
    const hrs = Math.max(1, Number(durationHours) || 12);
    const pacsPerHour = Math.floor(pacs / hrs);
    const remainder = pacs % hrs;

    let calculatedTotal = 0;
    const generatedHours = [];

    for (let i = 1; i <= hrs; i++) {
      const count = pacsPerHour + (i <= remainder ? 1 : 0);
      calculatedTotal += hourValue(count);
      generatedHours.push({ id: i, label: `Hora ${i}`, count });
    }

    const newShift = {
      id: Date.now(),
      date: dateStr,
      location: location || "Hospital Principal",
      type: type || `${hrs}h`,
      totalPatients: pacs,
      totalValue: calculatedTotal,
      hoursCount: hrs,
      hoursData: generatedHours,
      config: { ...config }
    };

    setHistory([newShift, ...history.filter(h => h.date !== dateStr)]);
    setSelectedDayModal(null);
  };

  const loadShiftToTracker = (shift) => {
    setShiftDate(shift.date);
    setShiftLocation(shift.location);
    setShiftType(shift.type);
    if (shift.hoursData && shift.hoursData.length > 0) {
      setHours(shift.hoursData);
      setNextId(shift.hoursData.length + 1);
    }
    setSelectedDayModal(null);
    setActiveTab("tracker");
  };

  const deleteShift = (id) => {
    if (window.confirm("Excluir este plantão do histórico?")) {
      setHistory(history.filter(h => h.id !== id));
      if (selectedDayModal) setSelectedDayModal(null);
    }
  };

  const updateConfig = (key, val) => {
    const n = Number(val);
    if (Number.isNaN(n) || n < 0) return;
    setConfig((c) => ({ ...c, [key]: n }));
  };

  const currentMonthShifts = history.filter(item => {
    const d = new Date(item.date + 'T00:00:00');
    return d.getFullYear() === viewDate.getFullYear() && d.getMonth() === viewDate.getMonth();
  });

  const monthTotalValue = currentMonthShifts.reduce((s, h) => s + h.totalValue, 0);
  const monthTotalPatients = currentMonthShifts.reduce((s, h) => s + h.totalPatients, 0);
  const monthTotalHours = currentMonthShifts.reduce((s, h) => s + h.hoursCount, 0);

  const avgShiftValue = currentMonthShifts.length > 0 ? monthTotalValue / currentMonthShifts.length : 0;
  
  const now = new Date();
  const isCurrentMonth = viewDate.getFullYear() === now.getFullYear() && viewDate.getMonth() === now.getMonth();
  const daysInSelectedMonth = new Date(viewDate.getFullYear(), viewDate.getMonth() + 1, 0).getDate();
  const daysPassed = isCurrentMonth ? now.getDate() : daysInSelectedMonth;
  const projectedValue = daysPassed > 0 ? (monthTotalValue / daysPassed) * daysInSelectedMonth : 0;

  const weeklyEarnings = [0, 0, 0, 0];
  currentMonthShifts.forEach(shift => {
    const day = new Date(shift.date + 'T00:00:00').getDate();
    if (day <= 7) weeklyEarnings[0] += shift.totalValue;
    else if (day <= 14) weeklyEarnings[1] += shift.totalValue;
    else if (day <= 21) weeklyEarnings[2] += shift.totalValue;
    else weeklyEarnings[3] += shift.totalValue;
  });
  const maxWeekly = Math.max(...weeklyEarnings, 1);

  let totalBaseVal = 0;
  let totalExtraVal = 0;
  currentMonthShifts.forEach(shift => {
    if (shift.hoursData) {
      shift.hoursData.forEach(h => {
        const c = h.count;
        const cfg = shift.config || config;
        if (c > 0) {
          const capped = Math.min(c, cfg.maxPatients);
          const baseCount = Math.min(capped, cfg.included);
          const extraCount = Math.max(0, capped - cfg.included);
          totalBaseVal += baseCount * cfg.baseValue;
          totalExtraVal += extraCount * cfg.extraValue;
        }
      });
    }
  });

  const changeMonth = (diff) => {
    setViewDate(new Date(viewDate.getFullYear(), viewDate.getMonth() + diff, 1));
  };

  const toggleExpandShift = (id) => {
    setExpandedShiftId(expandedShiftId === id ? null : id);
  };

  return (
    <div style={{ background: VARS.bg, color: VARS.text, minHeight: "100vh", fontFamily: "'Inter', sans-serif", paddingBottom: 40 }}>
      <div style={{
        position: "sticky", top: 0, zIndex: 10, background: "rgba(16,20,27,0.95)",
        backdropFilter: "blur(8px)", borderBottom: `1px solid ${VARS.border}`, padding: "12px 16px",
      }}>
        <div style={{ maxWidth: 480, margin: "0 auto" }}>
          <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 12 }}>
            <span style={{
              fontFamily: "'Space Grotesk', sans-serif", fontWeight: 700, fontSize: 15,
              letterSpacing: "0.08em", color: VARS.teal, textTransform: "uppercase",
            }}>Pulso · plantão</span>
            
            <div style={{ display: "flex", gap: 6 }}>
              {activeTab === "tracker" && (
                <button onClick={resetAll} title="Zerar plantão" style={iconBtnStyle()}>↺</button>
              )}
              <button onClick={() => setShowConfig((v) => !v)} title="Configuração" style={iconBtnStyle(showConfig)}>⚙</button>
            </div>
          </div>

          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 4, background: VARS.surface2, padding: 3, borderRadius: 10 }}>
            <button
              onClick={() => setActiveTab("tracker")}
              style={{
                padding: "8px 4px", borderRadius: 8, border: "none", cursor: "pointer", fontWeight: 600, fontSize: 11.5,
                background: activeTab === "tracker" ? VARS.surface : "transparent",
                color: activeTab === "tracker" ? VARS.teal : VARS.textMuted,
              }}
            >
              ⏱ Atual
            </button>
            <button
              onClick={() => setActiveTab("history")}
              style={{
                padding: "8px 4px", borderRadius: 8, border: "none", cursor: "pointer", fontWeight: 600, fontSize: 11.5,
                background: activeTab === "history" ? VARS.surface : "transparent",
                color: activeTab === "history" ? VARS.teal : VARS.textMuted,
              }}
            >
              📅 Histórico
            </button>
            <button
              onClick={() => setActiveTab("summary")}
              style={{
                padding: "8px 4px", borderRadius: 8, border: "none", cursor: "pointer", fontWeight: 600, fontSize: 11.5,
                background: activeTab === "summary" ? VARS.surface : "transparent",
                color: activeTab === "summary" ? VARS.teal : VARS.textMuted,
              }}
            >
              📊 Resumo
            </button>
          </div>
        </div>
      </div>

      {showConfig && (
        <div style={{ maxWidth: 480, margin: "0 auto", padding: "12px 16px 0" }}>
          <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 14, padding: 14 }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
              <span style={{ fontFamily: "'Space Grotesk', sans-serif", fontWeight: 600, fontSize: 13 }}>Regra de Pagamento</span>
              <button onClick={() => setShowConfig(false)} style={iconBtnStyle()}>✕</button>
            </div>
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10 }}>
              <ConfigField label="Valor / pac. (Base)" value={config.baseValue} onChange={(v) => updateConfig("baseValue", v)} />
              <ConfigField label="Pacientes Base" value={config.included} onChange={(v) => updateConfig("included", v)} />
              <ConfigField label="Valor Extra (R$)" value={config.extraValue} onChange={(v) => updateConfig("extraValue", v)} />
              <ConfigField label="Máx. p/ Hora" value={config.maxPatients} onChange={(v) => updateConfig("maxPatients", v)} />
            </div>
          </div>
        </div>
      )}

      {/* TAB 1: TRACKER */}
      {activeTab === "tracker" && (
        <div style={{ maxWidth: 480, margin: "0 auto", padding: "16px" }}>
          <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 16, padding: 16, marginBottom: 16 }}>
            <div style={{ display: "flex", alignItems: "flex-end", justifyContent: "space-between" }}>
              <div>
                <span style={{ fontSize: 10.5, color: VARS.textFaint, textTransform: "uppercase" }}>Faturamento Acumulado</span>
                <div style={{ fontFamily: "'JetBrains Mono', monospace", fontWeight: 700, fontSize: 34, color: VARS.teal, marginTop: 2 }}>
                  {formatBRL(totalValue)}
                </div>
              </div>
              <div style={{ textAlign: "right" }}>
                <div style={{ fontFamily: "'JetBrains Mono', monospace", fontSize: 15, fontWeight: 600, color: VARS.text }}>
                  {totalPatients} paciente{totalPatients === 1 ? "" : "s"}
                </div>
                <div style={{ fontSize: 11, color: VARS.textFaint, marginTop: 2 }}>
                  {formatBRL(avgPerHour)} / h · {hours.length} h
                </div>
              </div>
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "1.2fr 1fr 1fr", gap: 8, marginTop: 14, paddingTop: 12, borderTop: `1px solid ${VARS.borderSoft}` }}>
              <div>
                <span style={{ display: "block", fontSize: 10, color: VARS.teal, fontWeight: 700 }}>DATA</span>
                <input
                  type="date"
                  value={shiftDate}
                  onChange={e => setShiftDate(e.target.value)}
                  style={{ width: "100%", background: VARS.surface2, border: `1px solid ${VARS.border}`, borderRadius: 6, color: VARS.text, fontSize: 12, outline: "none", padding: "4px 6px" }}
                />
              </div>
              <div>
                <span style={{ display: "block", fontSize: 10, color: VARS.textFaint }}>LOCAL</span>
                <input
                  type="text"
                  value={shiftLocation}
                  onChange={e => setShiftLocation(e.target.value)}
                  style={{ width: "100%", background: "transparent", border: "none", color: VARS.text, fontSize: 12, outline: "none", fontWeight: 500 }}
                />
              </div>
              <div>
                <span style={{ display: "block", fontSize: 10, color: VARS.textFaint }}>TURNO</span>
                <input
                  type="text"
                  value={shiftType}
                  onChange={e => setShiftType(e.target.value)}
                  style={{ width: "100%", background: "transparent", border: "none", color: VARS.text, fontSize: 12, outline: "none", fontWeight: 500 }}
                />
              </div>
            </div>
          </div>

          {hours.map((h) => (
            <HourCard
              key={h.id}
              hour={h}
              config={config}
              value={hourValue(h.count)}
              onSet={(v) => setCount(h.id, v)}
              onRemove={() => removeHour(h.id)}
              canRemove={hours.length > 1}
            />
          ))}

          <button onClick={addHour} style={{
            width: "100%", padding: "12px", borderRadius: 12,
            border: `1.5px dashed ${VARS.border}`, background: "transparent", color: VARS.textMuted,
            fontWeight: 500, fontSize: 13.5, display: "flex", alignItems: "center", justifyContent: "center", gap: 6, cursor: "pointer", marginBottom: 16
          }}>
            + Adicionar Hora
          </button>

          <button onClick={saveShift} style={{
            width: "100%", padding: "14px", borderRadius: 12, border: "none",
            background: VARS.teal, color: "#0B1015", fontFamily: "'Space Grotesk', sans-serif",
            fontWeight: 700, fontSize: 15, cursor: "pointer", boxShadow: "0 4px 12px rgba(79, 209, 197, 0.2)"
          }}>
            ✓ Salvar Plantão no Histórico
          </button>
        </div>
      )}

      {/* TAB 2: HISTÓRICO */}
      {activeTab === "history" && (
        <div style={{ maxWidth: 480, margin: "0 auto", padding: "16px" }}>
          
          <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 16 }}>
            <button onClick={() => changeMonth(-1)} style={iconBtnStyle()}>◄</button>
            <span style={{ fontFamily: "'Space Grotesk', sans-serif", fontWeight: 700, fontSize: 16 }}>
              {viewDate.toLocaleDateString("pt-BR", { month: "long", year: "numeric" }).toUpperCase()}
            </span>
            <button onClick={() => changeMonth(1)} style={iconBtnStyle()}>►</button>
          </div>

          <CalendarView
            history={history}
            selectedYear={viewDate.getFullYear()}
            selectedMonth={viewDate.getMonth()}
            onSelectDay={(day, dateStr, shifts) => {
              setSelectedDayModal({ day, dateStr, shifts });
            }}
          />

          <h3 style={{ fontFamily: "'Space Grotesk', sans-serif", fontSize: 13, textTransform: "uppercase", color: VARS.textMuted, marginBottom: 10 }}>
            Plantões do Mês
          </h3>

          {currentMonthShifts.length === 0 ? (
            <div style={{ textAlign: "center", padding: "24px 10px", color: VARS.textFaint, border: `1px dashed ${VARS.border}`, borderRadius: 12 }}>
              Nenhum plantão registrado neste mês. Clique em um dia do calendário para adicionar.
            </div>
          ) : (
            currentMonthShifts.map((shift) => (
              <div key={shift.id} style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 14, padding: 14, marginBottom: 10 }}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                  <div>
                    <span style={{ fontSize: 11, color: VARS.teal, fontWeight: 600 }}>
                      {new Date(shift.date + 'T00:00:00').toLocaleDateString("pt-BR", { day: "2-digit", month: "2-digit", weekday: "short" })}
                    </span>
                    <h4 style={{ margin: "2px 0 0 0", fontSize: 15, fontWeight: 600 }}>{shift.location}</h4>
                    <span style={{ fontSize: 11, color: VARS.textFaint }}>{shift.type} · {shift.hoursCount}h · {shift.totalPatients} pacs</span>
                  </div>
                  <div style={{ textAlign: "right" }}>
                    <div style={{ fontFamily: "'JetBrains Mono', monospace", fontWeight: 700, fontSize: 16, color: VARS.teal }}>
                      {formatBRL(shift.totalValue)}
                    </div>
                    <div style={{ display: "flex", gap: 8, justifyContent: "flex-end", marginTop: 4 }}>
                      <button onClick={() => loadShiftToTracker(shift)} style={{ border: "none", background: "transparent", color: VARS.teal, fontSize: 11, cursor: "pointer", textDecoration: "underline" }}>
                        Editar
                      </button>
                      <button onClick={() => deleteShift(shift.id)} style={{ border: "none", background: "transparent", color: VARS.red, fontSize: 11, cursor: "pointer" }}>
                        Excluir
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            ))
          )}
        </div>
      )}

      {/* TAB 3: RESUMO MENSAL COM TABELA DE DETALHAMENTO */}
      {activeTab === "summary" && (
        <div style={{ maxWidth: 480, margin: "0 auto", padding: "16px" }}>
          
          <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 16 }}>
            <button onClick={() => changeMonth(-1)} style={iconBtnStyle()}>◄</button>
            <span style={{ fontFamily: "'Space Grotesk', sans-serif", fontWeight: 700, fontSize: 16 }}>
              {viewDate.toLocaleDateString("pt-BR", { month: "long", year: "numeric" }).toUpperCase()}
            </span>
            <button onClick={() => changeMonth(1)} style={iconBtnStyle()}>►</button>
          </div>

          {/* Banner Principal */}
          <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 16, padding: 16, marginBottom: 14 }}>
            <span style={{ fontSize: 10.5, color: VARS.textFaint, textTransform: "uppercase" }}>Faturamento Bruto no Mês</span>
            <div style={{ fontFamily: "'JetBrains Mono', monospace", fontWeight: 700, fontSize: 32, color: VARS.teal, marginTop: 2 }}>
              {formatBRL(monthTotalValue)}
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10, marginTop: 14, paddingTop: 12, borderTop: `1px solid ${VARS.borderSoft}` }}>
              <div>
                <span style={{ fontSize: 10, color: VARS.textFaint, display: "block" }}>MÉDIA / PLANTÃO</span>
                <span style={{ fontSize: 14, fontWeight: 700, color: VARS.text, fontFamily: "'JetBrains Mono', monospace" }}>
                  {formatBRL(avgShiftValue)}
                </span>
              </div>
              <div>
                <span style={{ fontSize: 10, color: VARS.textFaint, display: "block" }}>PROJEÇÃO ATÉ FIM DO MÊS</span>
                <span style={{ fontSize: 14, fontWeight: 700, color: VARS.amber, fontFamily: "'JetBrains Mono', monospace" }}>
                  {formatBRL(projectedValue)}
                </span>
              </div>
            </div>
          </div>

          {/* Cards de Métricas */}
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 8, marginBottom: 14 }}>
            <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 12, padding: 10, textAlign: "center" }}>
              <span style={{ fontSize: 9.5, color: VARS.textFaint, display: "block" }}>PLANTÕES</span>
              <span style={{ fontSize: 18, fontWeight: 700, color: VARS.text, fontFamily: "'JetBrains Mono', monospace" }}>{currentMonthShifts.length}</span>
            </div>
            <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 12, padding: 10, textAlign: "center" }}>
              <span style={{ fontSize: 9.5, color: VARS.textFaint, display: "block" }}>HORAS TRABALHADAS</span>
              <span style={{ fontSize: 18, fontWeight: 700, color: VARS.text, fontFamily: "'JetBrains Mono', monospace" }}>{monthTotalHours}h</span>
            </div>
            <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 12, padding: 10, textAlign: "center" }}>
              <span style={{ fontSize: 9.5, color: VARS.textFaint, display: "block" }}>TOTAL DE PACS</span>
              <span style={{ fontSize: 18, fontWeight: 700, color: VARS.amber, fontFamily: "'JetBrains Mono', monospace" }}>{monthTotalPatients}</span>
            </div>
          </div>

          {/* NOVO: TABELA DE PLANTÕES EXPANSÍVEL (ACCORDION) */}
          <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 16, overflow: "hidden", marginBottom: 14 }}>
            <div style={{ padding: "14px 16px", borderBottom: `1px solid ${VARS.border}`, display: "flex", justifyContent: "space-between", alignItems: "center" }}>
              <h4 style={{ margin: 0, fontSize: 12, textTransform: "uppercase", color: VARS.textMuted, fontFamily: "'Space Grotesk', sans-serif" }}>
                Tabela Detalhada de Plantões
              </h4>
              <span style={{ fontSize: 10, color: VARS.textFaint }}>Clique para expandir</span>
            </div>

            {currentMonthShifts.length === 0 ? (
              <div style={{ padding: 20, textAlign: "center", fontSize: 12, color: VARS.textFaint }}>
                Nenhum dado para exibir neste mês.
              </div>
            ) : (
              <div style={{ overflowX: "auto" }}>
                <table style={{ width: "100%", borderCollapse: "collapse", textAlign: "left", fontSize: 12 }}>
                  <thead>
                    <tr style={{ background: VARS.surface2, borderBottom: `1px solid ${VARS.border}`, color: VARS.textFaint, fontSize: 10.5 }}>
                      <th style={{ padding: "10px 12px" }}>DATA</th>
                      <th style={{ padding: "10px 8px" }}>LOCAL</th>
                      <th style={{ padding: "10px 8px", textAlign: "center" }}>PACS</th>
                      <th style={{ padding: "10px 12px", textAlign: "right" }}>VALOR</th>
                    </tr>
                  </thead>
                  <tbody>
                    {currentMonthShifts.map((shift) => {
                      const isExpanded = expandedShiftId === shift.id;
                      const d = new Date(shift.date + 'T00:00:00');
                      const dateFormatted = `${String(d.getDate()).padStart(2, '0')}/${String(d.getMonth() + 1).padStart(2, '0')}`;

                      return (
                        <React.Fragment key={shift.id}>
                          <tr
                            onClick={() => toggleExpandShift(shift.id)}
                            style={{
                              borderBottom: isExpanded ? "none" : `1px solid ${VARS.borderSoft}`,
                              background: isExpanded ? VARS.tealSoft : "transparent",
                              cursor: "pointer",
                              transition: "background 0.15s ease"
                            }}
                          >
                            <td style={{ padding: "12px", fontWeight: 600, color: VARS.teal }}>
                              {dateFormatted}
                            </td>
                            <td style={{ padding: "12px 8px", color: VARS.text }}>
                              <div style={{ fontWeight: 600, fontSize: 12.5 }}>{shift.location}</div>
                              <div style={{ fontSize: 10, color: VARS.textFaint }}>{shift.type}</div>
                            </td>
                            <td style={{ padding: "12px 8px", textAlign: "center", fontFamily: "'JetBrains Mono', monospace", fontWeight: 600, color: VARS.amber }}>
                              {shift.totalPatients}
                            </td>
                            <td style={{ padding: "12px", textAlign: "right", fontFamily: "'JetBrains Mono', monospace", fontWeight: 700, color: VARS.teal }}>
                              {formatBRL(shift.totalValue)}
                            </td>
                          </tr>

                          {/* Conteúdo Detalhado ao Clicar */}
                          {isExpanded && (
                            <tr style={{ background: VARS.surface2, borderBottom: `1px solid ${VARS.border}` }}>
                              <td colSpan="4" style={{ padding: "12px 14px" }}>
                                <div style={{ fontSize: 11, fontWeight: 700, color: VARS.teal, marginBottom: 8, textTransform: "uppercase" }}>
                                  🔎 Atendimentos Hora a Hora ({shift.hoursCount}h)
                                </div>
                                <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fill, minmax(130px, 1fr))", gap: 6 }}>
                                  {shift.hoursData && shift.hoursData.map((h) => (
                                    <div key={h.id} style={{ background: VARS.surface, border: `1px solid ${VARS.borderSoft}`, borderRadius: 6, padding: "6px 8px", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                                      <span style={{ fontSize: 10.5, color: VARS.textMuted }}>{h.label}:</span>
                                      <span style={{ fontSize: 11, fontFamily: "'JetBrains Mono', monospace", fontWeight: 700, color: h.count > config.included ? VARS.amber : VARS.text }}>
                                        {h.count} pac{h.count === 1 ? '' : 's'}
                                      </span>
                                    </div>
                                  ))}
                                </div>
                              </td>
                            </tr>
                          )}
                        </React.Fragment>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            )}
          </div>

          {/* Gráfico Semanal */}
          <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 16, padding: 16, marginBottom: 14 }}>
            <h4 style={{ margin: "0 0 12px 0", fontSize: 12, textTransform: "uppercase", color: VARS.textMuted, fontFamily: "'Space Grotesk', sans-serif" }}>
              Distribuição Semanal de Ganhos
            </h4>
            <div style={{ display: "flex", alignItems: "flex-end", gap: 10, height: 90, paddingTop: 10 }}>
              {weeklyEarnings.map((val, idx) => {
                const heightPct = Math.max(8, Math.round((val / maxWeekly) * 100));
                return (
                  <div key={idx} style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", gap: 6, height: "100%", justifyContent: "flex-end" }}>
                    <span style={{ fontSize: 9, fontFamily: "'JetBrains Mono', monospace", color: VARS.teal }}>
                      {val > 0 ? `R$${Math.round(val)}` : ''}
                    </span>
                    <div style={{ width: "100%", height: `${heightPct}%`, background: val > 0 ? VARS.teal : VARS.surface2, border: `1px solid ${val > 0 ? VARS.teal : VARS.borderSoft}`, borderRadius: 6, transition: "all 0.3s ease" }} />
                    <span style={{ fontSize: 10, color: VARS.textFaint }}>Sem {idx + 1}</span>
                  </div>
                );
              })}
            </div>
          </div>

          {/* Composição do Faturamento */}
          <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 16, padding: 16 }}>
            <h4 style={{ margin: "0 0 12px 0", fontSize: 12, textTransform: "uppercase", color: VARS.textMuted, fontFamily: "'Space Grotesk', sans-serif" }}>
              Origem dos Rendimentos
            </h4>
            
            <div style={{ marginBottom: 10 }}>
              <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, marginBottom: 4 }}>
                <span style={{ color: VARS.text }}>Faturamento Base (Cota Fixa)</span>
                <span style={{ fontFamily: "'JetBrains Mono', monospace", fontWeight: 600 }}>{formatBRL(totalBaseVal)}</span>
              </div>
              <div style={{ width: "100%", height: 6, background: VARS.surface2, borderRadius: 3, overflow: "hidden" }}>
                <div style={{ width: `${monthTotalValue > 0 ? (totalBaseVal / monthTotalValue) * 100 : 0}%`, height: "100%", background: VARS.teal }} />
              </div>
            </div>

            <div>
              <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, marginBottom: 4 }}>
                <span style={{ color: VARS.amber }}>Bônus Pacientes Extras</span>
                <span style={{ fontFamily: "'JetBrains Mono', monospace", fontWeight: 600, color: VARS.amber }}>{formatBRL(totalExtraVal)}</span>
              </div>
              <div style={{ width: "100%", height: 6, background: VARS.surface2, borderRadius: 3, overflow: "hidden" }}>
                <div style={{ width: `${monthTotalValue > 0 ? (totalExtraVal / monthTotalValue) * 100 : 0}%`, height: "100%", background: VARS.amber }} />
              </div>
            </div>
          </div>

        </div>
      )}

      {/* MODAL DE EDIÇÃO E LANÇAMENTO RETROATIVO */}
      {selectedDayModal && (
        <div style={{
          position: "fixed", top: 0, left: 0, right: 0, bottom: 0, background: "rgba(0,0,0,0.8)",
          display: "flex", alignItems: "center", justifyContent: "center", padding: 16, zIndex: 100
        }}>
          <div style={{ background: VARS.surface, border: `1px solid ${VARS.border}`, borderRadius: 16, padding: 18, width: "100%", maxWidth: 420 }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 14 }}>
              <h3 style={{ margin: 0, fontSize: 15, fontFamily: "'Space Grotesk', sans-serif" }}>
                Dia {new Date(selectedDayModal.dateStr + 'T00:00:00').toLocaleDateString("pt-BR", { day: "2-digit", month: "long", year: "numeric" })}
              </h3>
              <button onClick={() => setSelectedDayModal(null)} style={iconBtnStyle()}>✕</button>
            </div>

            {selectedDayModal.shifts.length > 0 ? (
              <div>
                <p style={{ fontSize: 12, color: VARS.teal, marginBottom: 12, fontWeight: 600 }}>
                  Existe 1 plantão salvo nesta data:
                </p>
                {selectedDayModal.shifts.map(s => (
                  <div key={s.id} style={{ background: VARS.surface2, border: `1px solid ${VARS.border}`, borderRadius: 10, padding: 12, marginBottom: 12 }}>
                    <div style={{ fontWeight: 600, fontSize: 14 }}>{s.location}</div>
                    <div style={{ fontSize: 11, color: VARS.textMuted }}>{s.type} · {s.totalPatients} pacientes · {formatBRL(s.totalValue)}</div>
                    <div style={{ display: "flex", gap: 8, marginTop: 10 }}>
                      <button onClick={() => loadShiftToTracker(s)} style={{ flex: 1, padding: "8px", borderRadius: 6, border: "none", background: VARS.teal, color: "#000", fontWeight: 700, fontSize: 12, cursor: "pointer" }}>
                        Editar no Contador
                      </button>
                      <button onClick={() => deleteShift(s.id)} style={{ padding: "8px", borderRadius: 6, border: `1px solid ${VARS.red}`, background: "transparent", color: VARS.red, fontWeight: 600, fontSize: 12, cursor: "pointer" }}>
                        Excluir
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            ) : (
              <div>
                <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 6, background: VARS.surface2, padding: 3, borderRadius: 8, marginBottom: 14 }}>
                  <button
                    onClick={() => setEditMode("quick")}
                    style={{ padding: "6px", borderRadius: 6, border: "none", cursor: "pointer", fontSize: 11.5, fontWeight: 600, background: editMode === "quick" ? VARS.surface : "transparent", color: editMode === "quick" ? VARS.teal : VARS.textMuted }}
                  >
                    ⚡ Lançamento Rápido
                  </button>
                  <button
                    onClick={() => setEditMode("tracker")}
                    style={{ padding: "6px", borderRadius: 6, border: "none", cursor: "pointer", fontSize: 11.5, fontWeight: 600, background: editMode === "tracker" ? VARS.surface : "transparent", color: editMode === "tracker" ? VARS.teal : VARS.textMuted }}
                  >
                    ⏱ Abrir no Contador
                  </button>
                </div>

                {editMode === "quick" ? (
                  <form onSubmit={(e) => {
                    e.preventDefault();
                    saveQuickShift(
                      selectedDayModal.dateStr,
                      e.target.loc.value,
                      e.target.type.value,
                      e.target.pacs.value,
                      e.target.hrs.value
                    );
                  }}>
                    <div style={{ display: "flex", flexDirection: "column", gap: 10, marginBottom: 14 }}>
                      <div>
                        <span style={{ display: "block", fontSize: 10.5, color: VARS.textFaint, marginBottom: 3 }}>LOCAL DO PLANTÃO</span>
                        <input name="loc" defaultValue="Hospital Principal" style={{ width: "100%", padding: "8px 10px", background: VARS.surface2, border: `1px solid ${VARS.border}`, borderRadius: 8, color: VARS.text, fontSize: 13 }} required />
                      </div>
                      
                      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 8 }}>
                        <div>
                          <span style={{ display: "block", fontSize: 10.5, color: VARS.textFaint, marginBottom: 3 }}>TURNO</span>
                          <input name="type" defaultValue="12h" style={{ width: "100%", padding: "8px 10px", background: VARS.surface2, border: `1px solid ${VARS.border}`, borderRadius: 8, color: VARS.text, fontSize: 13 }} required />
                        </div>
                        <div>
                          <span style={{ display: "block", fontSize: 10.5, color: VARS.textFaint, marginBottom: 3 }}>DURAÇÃO (HORAS)</span>
                          <input name="hrs" type="number" defaultValue="12" style={{ width: "100%", padding: "8px 10px", background: VARS.surface2, border: `1px solid ${VARS.border}`, borderRadius: 8, color: VARS.text, fontSize: 13 }} required />
                        </div>
                      </div>

                      <div>
                        <span style={{ display: "block", fontSize: 10.5, color: VARS.teal, fontWeight: 700, marginBottom: 3 }}>TOTAL DE PACIENTES ATENDIDOS</span>
                        <input name="pacs" type="number" placeholder="Ex: 35" style={{ width: "100%", padding: "8px 10px", background: VARS.surface2, border: `1px solid ${VARS.teal}`, borderRadius: 8, color: VARS.teal, fontFamily: "'JetBrains Mono', monospace", fontWeight: 700, fontSize: 15 }} required />
                      </div>
                    </div>

                    <button type="submit" style={{ width: "100%", padding: "12px", borderRadius: 10, border: "none", background: VARS.teal, color: "#000", fontWeight: 700, fontSize: 13, cursor: "pointer" }}>
                      Calcular e Salvar no Histórico
                    </button>
                  </form>
                ) : (
                  <div>
                    <p style={{ fontSize: 12, color: VARS.textMuted, marginBottom: 12 }}>
                      Carrega a aba do contador configurada nesta data para você preencher paciente por paciente por hora.
                    </p>
                    <button
                      onClick={() => {
                        setShiftDate(selectedDayModal.dateStr);
                        setSelectedDayModal(null);
                        setActiveTab("tracker");
                      }}
                      style={{ width: "100%", padding: "12px", borderRadius: 10, border: "none", background: VARS.teal, color: "#000", fontWeight: 700, fontSize: 13, cursor: "pointer" }}
                    >
                      Ir para Lançamento Hora a Hora
                    </button>
                  </div>
                )}
              </div>
            )}
          </div>
        </div>
      )}
    </div>
  );
}

ReactDOM.createRoot(document.getElementById("root")).render(<PlantaoTracker />);
</script>
</body>
</html>
