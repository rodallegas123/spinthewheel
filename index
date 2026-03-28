import { useState, useEffect, useRef, useCallback } from "react";

const GOOGLE_REVIEW_URL = "https://g.page/r/YOUR_GOOGLE_REVIEW_LINK";
const STORE_NAME        = "P.S. I Crepe You";

// ─── 12 segments — heavier on 10% and 20% ────────────────────────────────────
const PRIZES = [
  { label: "10% Off",     emoji: "🏷️", color: ["#f9a8c9","#e8457a"], weight: 14, tier: 1 },
  { label: "20% Off",     emoji: "✂️",  color: ["#f06292","#c2185b"], weight: 10, tier: 2 },
  { label: "10% Off",     emoji: "🏷️", color: ["#f9a8c9","#e8457a"], weight: 12, tier: 1 },
  { label: "Free Coffee", emoji: "☕",  color: ["#ba68c8","#7b1fa2"], weight: 11, tier: 2 },
  { label: "20% Off",     emoji: "✂️",  color: ["#f06292","#c2185b"], weight: 10, tier: 2 },
  { label: "10% Off",     emoji: "🏷️", color: ["#f9a8c9","#e8457a"], weight: 12, tier: 1 },
  { label: "30% Off",     emoji: "🎉",  color: ["#e57373","#c62828"], weight: 11, tier: 3 },
  { label: "20% Off",     emoji: "✂️",  color: ["#f06292","#c2185b"], weight:  9, tier: 2 },
  { label: "10% Off",     emoji: "🏷️", color: ["#f9a8c9","#e8457a"], weight:  9, tier: 1 },
  { label: "40% Off",     emoji: "🔥",  color: ["#ff7043","#bf360c"], weight:  6, tier: 3 },
  { label: "50% Off",     emoji: "💥",  color: ["#ffd54f","#f57f17"], weight:  4, tier: 4 },
  { label: "Free Crepe",  emoji: "🥞",  color: ["#ce93d8","#4a148c"], weight:  2, tier: 4 },
];

const SEG_COUNT = PRIZES.length;
const SEG_ANGLE = (Math.PI * 2) / SEG_COUNT;
const PINK  = "#e8457a";
const DPINK = "#c2185b";
const LIGHT = "#fce4ec";

function pickPrize() {
  const total = PRIZES.reduce((s, p) => s + p.weight, 0);
  let r = Math.random() * total;
  for (let i = 0; i < PRIZES.length; i++) { r -= PRIZES[i].weight; if (r <= 0) return i; }
  return 0;
}

function genCode(label) {
  const tag = label.replace(/[^a-zA-Z]/g,"").toUpperCase().slice(0,5);
  return `${tag}-${Math.random().toString(36).substring(2,5).toUpperCase()}`;
}

function hex2rgb(hex) {
  const n = parseInt(hex.replace("#",""), 16);
  return [(n>>16)&255, (n>>8)&255, n&255];
}

// ─── HIGH-QUALITY WHEEL ───────────────────────────────────────────────────────
function SpinWheel({ spinning, targetIndex, onDone, idle, size = 600 }) {
  const canvasRef  = useRef(null);
  const angleRef   = useRef(-Math.PI / 2); // start so first segment at top
  const rafRef     = useRef(null);
  const idleRef    = useRef(null);
  const R          = size / 2 - 10;
  const cx = size / 2, cy = size / 2;

  const draw = useCallback((angle) => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    ctx.clearRect(0, 0, size, size);

    // ── outer decorative ring ──
    ctx.save();
    const ringGrad = ctx.createLinearGradient(0,0,size,size);
    ringGrad.addColorStop(0, "#ffc0d4");
    ringGrad.addColorStop(0.5, "#fff");
    ringGrad.addColorStop(1, "#f48fb1");
    ctx.beginPath();
    ctx.arc(cx, cy, R + 9, 0, Math.PI*2);
    ctx.strokeStyle = ringGrad;
    ctx.lineWidth = 14;
    ctx.stroke();
    ctx.restore();

    // ── gold studs on outer ring ──
    for (let i = 0; i < 36; i++) {
      const a = (i / 36) * Math.PI * 2 + angle * 0.05;
      const sx = cx + Math.cos(a) * (R + 9);
      const sy = cy + Math.sin(a) * (R + 9);
      ctx.beginPath();
      ctx.arc(sx, sy, 3.5, 0, Math.PI*2);
      const studG = ctx.createRadialGradient(sx-1,sy-1,0,sx,sy,3.5);
      studG.addColorStop(0,"#fff");
      studG.addColorStop(1,"#f48fb1");
      ctx.fillStyle = studG;
      ctx.fill();
    }

    // ── drop shadow under wheel ──
    ctx.save();
    ctx.shadowColor = "rgba(232,69,122,0.35)";
    ctx.shadowBlur  = 40;
    ctx.shadowOffsetY = 12;
    ctx.beginPath();
    ctx.arc(cx, cy, R, 0, Math.PI*2);
    ctx.fillStyle = "#fff";
    ctx.fill();
    ctx.restore();

    // ── segments ──
    PRIZES.forEach((p, i) => {
      const start = angle + i * SEG_ANGLE;
      const end   = start + SEG_ANGLE;
      const midA  = start + SEG_ANGLE / 2;

      // radial gradient per segment
      const gx = cx + Math.cos(midA) * R * 0.5;
      const gy = cy + Math.sin(midA) * R * 0.5;
      const g  = ctx.createRadialGradient(gx, gy, 0, gx, gy, R * 0.85);
      g.addColorStop(0, p.color[0]);
      g.addColorStop(1, p.color[1]);

      ctx.beginPath();
      ctx.moveTo(cx, cy);
      ctx.arc(cx, cy, R, start, end);
      ctx.closePath();
      ctx.fillStyle = g;
      ctx.fill();

      // sheen overlay
      ctx.save();
      ctx.clip();
      ctx.beginPath();
      ctx.moveTo(cx, cy);
      ctx.arc(cx, cy, R, start, end);
      ctx.closePath();
      const sheen = ctx.createLinearGradient(
        cx + Math.cos(start) * R * 0.2, cy + Math.sin(start) * R * 0.2,
        cx + Math.cos(end)   * R * 0.8, cy + Math.sin(end)   * R * 0.8,
      );
      sheen.addColorStop(0, "rgba(255,255,255,0.18)");
      sheen.addColorStop(0.5,"rgba(255,255,255,0)");
      sheen.addColorStop(1, "rgba(0,0,0,0.08)");
      ctx.fillStyle = sheen;
      ctx.fill();
      ctx.restore();

      // divider lines
      ctx.beginPath();
      ctx.moveTo(cx, cy);
      ctx.lineTo(cx + Math.cos(start) * R, cy + Math.sin(start) * R);
      ctx.strokeStyle = "rgba(255,255,255,0.4)";
      ctx.lineWidth = 2;
      ctx.stroke();

      // emoji + label
      ctx.save();
      ctx.translate(cx, cy);
      ctx.rotate(midA);
      ctx.textAlign = "right";

      // text shadow
      ctx.shadowColor = "rgba(0,0,0,0.35)";
      ctx.shadowBlur  = 5;

      ctx.font = `bold ${size < 400 ? 11 : 16}px Georgia,serif`;
      ctx.fillStyle = "#fff";
      ctx.fillText(p.label, R - 22, 7);

      ctx.font = `${size < 400 ? 16 : 22}px serif`;
      ctx.fillText(p.emoji, R - 22, -12);
      ctx.restore();
    });

    // ── inner shadow ring ──
    const innerShadow = ctx.createRadialGradient(cx,cy,R*0.7,cx,cy,R);
    innerShadow.addColorStop(0,"rgba(0,0,0,0)");
    innerShadow.addColorStop(1,"rgba(0,0,0,0.12)");
    ctx.beginPath();
    ctx.arc(cx,cy,R,0,Math.PI*2);
    ctx.fillStyle = innerShadow;
    ctx.fill();

    // ── metallic center hub ──
    const hubR  = size < 400 ? 30 : 48;
    const hubG  = ctx.createRadialGradient(cx-hubR*0.3, cy-hubR*0.3, 0, cx, cy, hubR);
    hubG.addColorStop(0,"#fff");
    hubG.addColorStop(0.4,"#fce4ec");
    hubG.addColorStop(0.8,"#f8bbd0");
    hubG.addColorStop(1,"#e8457a");
    ctx.save();
    ctx.shadowColor = "rgba(0,0,0,0.3)";
    ctx.shadowBlur  = 16;
    ctx.beginPath();
    ctx.arc(cx,cy,hubR,0,Math.PI*2);
    ctx.fillStyle = hubG;
    ctx.fill();
    ctx.restore();

    // hub ring
    ctx.beginPath();
    ctx.arc(cx,cy,hubR,0,Math.PI*2);
    ctx.strokeStyle = "#fff";
    ctx.lineWidth = 3;
    ctx.stroke();
    ctx.beginPath();
    ctx.arc(cx,cy,hubR-3,0,Math.PI*2);
    ctx.strokeStyle = PINK;
    ctx.lineWidth = 2;
    ctx.stroke();

    // brand text in hub
    ctx.fillStyle = DPINK;
    ctx.textAlign = "center";
    ctx.font = `bold ${size < 400 ? 8 : 11}px Georgia,serif`;
    ctx.shadowColor = "transparent";
    ctx.fillText("P.S.I", cx, cy - 3);
    ctx.fillText("CREPE", cx, cy + 11);

  }, [size]);

  // idle slow drift
  useEffect(() => {
    if (!idle) { cancelAnimationFrame(idleRef.current); return; }
    const drift = () => {
      angleRef.current += 0.004;
      draw(angleRef.current);
      idleRef.current = requestAnimationFrame(drift);
    };
    idleRef.current = requestAnimationFrame(drift);
    return () => cancelAnimationFrame(idleRef.current);
  }, [idle, draw]);

  useEffect(() => { if (!idle) draw(angleRef.current); }, [draw, idle]);

  // spin
  useEffect(() => {
    if (!spinning) return;
    cancelAnimationFrame(idleRef.current);

    // target: center of targetIndex segment at top (angle = -PI/2)
    const targetMid = -(targetIndex * SEG_ANGLE + SEG_ANGLE / 2) - Math.PI/2;
    const spins     = (7 + Math.random() * 4) * Math.PI * 2;
    const start0    = angleRef.current;
    const delta     = spins + ((targetMid - start0) % (Math.PI * 2) + Math.PI * 2) % (Math.PI * 2);
    const duration  = 5500;
    const t0        = performance.now();

    const animate = (now) => {
      const t    = Math.min((now - t0) / duration, 1);
      const ease = t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2,3)/2;
      angleRef.current = start0 + delta * ease;
      draw(angleRef.current);
      if (t < 1) { rafRef.current = requestAnimationFrame(animate); }
      else        { onDone(); }
    };
    rafRef.current = requestAnimationFrame(animate);
    return () => cancelAnimationFrame(rafRef.current);
  }, [spinning, targetIndex, draw, onDone]);

  return (
    <canvas ref={canvasRef} width={size} height={size}
      style={{ display:"block", maxWidth:"100%", maxHeight:"100%",
               filter: spinning ? "drop-shadow(0 0 32px rgba(232,69,122,0.7))" : "drop-shadow(0 8px 24px rgba(232,69,122,0.3))",
               transition:"filter 0.5s" }} />
  );
}

// ─── POINTER ──────────────────────────────────────────────────────────────────
function Pointer({ size }) {
  const s = size || 32;
  return (
    <svg width={s} height={s * 1.4} viewBox="0 0 32 44" style={{ filter:"drop-shadow(0 2px 6px rgba(232,69,122,0.8))" }}>
      <defs>
        <linearGradient id="pg" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stopColor="#fff"/>
          <stop offset="100%" stopColor="#f48fb1"/>
        </linearGradient>
      </defs>
      <polygon points="16,40 3,4 29,4" fill="#e8457a" stroke="#fff" strokeWidth="2"/>
      <polygon points="16,36 6,8 26,8" fill="url(#pg)" opacity="0.5"/>
      <circle cx="16" cy="7" r="5" fill="#fff" stroke="#e8457a" strokeWidth="2"/>
      <circle cx="16" cy="7" r="2.5" fill="#e8457a"/>
    </svg>
  );
}

// ─── CONFETTI ─────────────────────────────────────────────────────────────────
function Confetti() {
  const ref = useRef(null);
  useEffect(() => {
    const c = ref.current;
    const ctx = c.getContext("2d");
    c.width = window.innerWidth; c.height = window.innerHeight;
    const shapes = ["rect","circle","star"];
    const pieces = Array.from({length:140}, () => ({
      x: Math.random()*c.width, y:-40-Math.random()*200,
      r: 5+Math.random()*9,
      col:["#e8457a","#f48fb1","#fce4ec","#c2185b","#fff","#fbbf24","#ce93d8","#ff7043"][Math.floor(Math.random()*8)],
      vx:-3+Math.random()*6, vy:2+Math.random()*6,
      ang:Math.random()*360, spin:-5+Math.random()*10,
      shape:shapes[Math.floor(Math.random()*3)], opacity:1,
    }));
    let raf; let elapsed=0; const start=performance.now();
    const tick=(now)=>{
      elapsed = now-start;
      ctx.clearRect(0,0,c.width,c.height);
      const fade = Math.max(0, 1-(elapsed-3000)/1500);
      pieces.forEach(p=>{
        ctx.save();
        ctx.globalAlpha = p.opacity * fade;
        ctx.translate(p.x,p.y); ctx.rotate(p.ang*Math.PI/180);
        ctx.fillStyle=p.col;
        if(p.shape==="rect"){ ctx.fillRect(-p.r,-p.r/2,p.r*2,p.r); }
        else if(p.shape==="circle"){ ctx.beginPath();ctx.arc(0,0,p.r/2,0,Math.PI*2);ctx.fill(); }
        else {
          ctx.beginPath();
          for(let s=0;s<5;s++){
            const a=s*Math.PI*2/5-Math.PI/2;
            const a2=a+Math.PI/5;
            if(s===0)ctx.moveTo(Math.cos(a)*p.r,Math.sin(a)*p.r);
            else ctx.lineTo(Math.cos(a)*p.r,Math.sin(a)*p.r);
            ctx.lineTo(Math.cos(a2)*p.r*0.4,Math.sin(a2)*p.r*0.4);
          }
          ctx.closePath(); ctx.fill();
        }
        ctx.restore();
        p.x+=p.vx; p.y+=p.vy; p.ang+=p.spin; p.vy+=0.08;
      });
      if(elapsed<4500) raf=requestAnimationFrame(tick);
    };
    raf=requestAnimationFrame(tick);
    return ()=>cancelAnimationFrame(raf);
  },[]);
  return <canvas ref={ref} style={{position:"fixed",inset:0,pointerEvents:"none",zIndex:300}}/>;
}

// ─── SPARKLE BACKGROUND ───────────────────────────────────────────────────────
function SparklesBg() {
  return (
    <svg style={{position:"absolute",inset:0,width:"100%",height:"100%",pointerEvents:"none",zIndex:0}} xmlns="http://www.w3.org/2000/svg">
      {Array.from({length:22},(_,i)=>{
        const x=`${5+Math.sin(i*1.7)*45+50}%`;
        const y=`${5+Math.cos(i*2.3)*45+50}%`;
        const s=0.4+Math.sin(i)*0.6;
        const d=i*0.4;
        return (
          <g key={i} style={{animation:`sparkle ${2+Math.random()*2}s ${d}s ease-in-out infinite`}}>
            <circle cx={x} cy={y} r={`${3*s}`} fill={i%3===0?"#e8457a":i%3===1?"#fce4ec":"#c2185b"} opacity="0.25"/>
          </g>
        );
      })}
    </svg>
  );
}

// ─── PHONE / CONSENT MODAL ────────────────────────────────────────────────────
function Modal({ onConfirm, onClose }) {
  const [phone, setPhone] = useState("");
  const [agreed, setAgreed] = useState(false);
  const [err, setErr] = useState("");

  const fmt = (v) => {
    const d=v.replace(/\D/g,"").slice(0,10);
    if(d.length<=3)return d;
    if(d.length<=6)return `(${d.slice(0,3)}) ${d.slice(3)}`;
    return `(${d.slice(0,3)}) ${d.slice(3,6)}-${d.slice(6)}`;
  };

  const submit = () => {
    if(phone.replace(/\D/g,"").length<10){setErr("Please enter a valid 10-digit number.");return;}
    if(!agreed){setErr("Please agree to receive your prize by text.");return;}
    onConfirm(phone.replace(/\D/g,""));
  };

  return (
    <div style={MO.overlay} onClick={onClose}>
      <div style={MO.modal} onClick={e=>e.stopPropagation()} className="modal-in">
        <button style={MO.close} onClick={onClose}>✕</button>
        <div style={MO.icon}>🎰</div>
        <div style={MO.title}>One quick step!</div>
        <div style={MO.sub}>Enter your number and we'll text you your prize + a link to share the love. 💕</div>
        <input
          style={{...MO.input,...(err&&!phone?MO.inputErr:{})}}
          type="tel" inputMode="numeric" placeholder="(951) 555-0000"
          value={phone} onChange={e=>{setPhone(fmt(e.target.value));setErr("");}}
          autoFocus
        />
        <div style={{...MO.row,borderColor:agreed?PINK:"#f8bbd0"}} onClick={()=>{setAgreed(!agreed);setErr("");}}>
          <div style={{...MO.cb,...(agreed?MO.cbOn:{})}}>
            {agreed&&<span style={{color:"#fff",fontSize:16}}>✓</span>}
          </div>
          <div style={{textAlign:"left"}}>
            <div style={MO.cbLabel}>Text me my prize &amp; deals 🎁</div>
            <div style={MO.cbSub}>Max 2 msgs/month · Reply STOP anytime</div>
          </div>
        </div>
        {err&&<div style={MO.err}>{err}</div>}
        <div style={MO.legal}>By tapping Spin, you consent to receive marketing SMS from {STORE_NAME}. Msg &amp; data rates may apply.</div>
        <button style={MO.btn} onClick={submit}>🎰 Spin the Wheel!</button>
      </div>
    </div>
  );
}

// ─── APP ──────────────────────────────────────────────────────────────────────
const SCREEN = {HOME:"home",SPIN:"spin",RESULT:"result"};

export default function App() {
  const [screen,    setScreen]    = useState(SCREEN.HOME);
  const [modal,     setModal]     = useState(false);
  const [spinning,  setSpinning]  = useState(false);
  const [targetIdx, setTargetIdx] = useState(0);
  const [prize,     setPrize]     = useState(null);
  const [code,      setCode]      = useState("");
  const [confetti,  setConfetti]  = useState(false);
  const [wheelSize, setWheelSize] = useState(580);
  const [played,    setPlayed]    = useState(()=>{
    try{return JSON.parse(localStorage.getItem("psicy_v3")||"{}");}catch{return{};}
  });

  // responsive wheel size
  useEffect(()=>{
    const set=()=>{
      const s=Math.min(window.innerWidth,window.innerHeight);
      setWheelSize(Math.min(580, Math.max(300, s - 240)));
    };
    set(); window.addEventListener("resize",set);
    return ()=>window.removeEventListener("resize",set);
  },[]);

  const handleSpin = (digits) => {
    const today=new Date().toDateString();
    if(played[digits]===today){ alert("You already spun today! Come back next visit 😄"); setModal(false); return; }
    setModal(false);
    const idx=pickPrize();
    setTargetIdx(idx);
    const updated={...played,[digits]:today};
    setPlayed(updated);
    localStorage.setItem("psicy_v3",JSON.stringify(updated));
    // SMS hook (replace with real backend)
    console.log(`[SMS→${digits}] Thanks for spinning! 🥞 Leave us a review: ${GOOGLE_REVIEW_URL}`);
    setScreen(SCREEN.SPIN);
    setTimeout(()=>setSpinning(true),400);
  };

  const onSpinDone = useCallback(()=>{
    setSpinning(false);
    const p=PRIZES[targetIdx];
    setPrize(p);
    setCode(genCode(p.label));
    setConfetti(true);
    setTimeout(()=>setConfetti(false),5000);
    setTimeout(()=>setScreen(SCREEN.RESULT),700);
  },[targetIdx]);

  const reset=()=>{ setScreen(SCREEN.HOME); setPrize(null); setCode(""); setSpinning(false); };

  const tierAccent=(tier)=>["","#e8457a","#c2185b","#bf360c","#7b1fa2"][tier]||PINK;

  return (
    <div style={A.root}>
      <style>{CSS}</style>
      {confetti&&<Confetti/>}
      {modal&&<Modal onConfirm={handleSpin} onClose={()=>setModal(false)}/>}

      {/* ── HOME ── */}
      {screen===SCREEN.HOME&&(
        <div style={A.home} className="fade-in">
          <SparklesBg/>

          {/* Header */}
          <div style={A.header}>
            <div style={A.headerInner}>
              <div>
                <div style={A.brand}>P.S. I Crepe You</div>
                <div style={A.brandSub}>Riverside, CA · Fresh Crêpes</div>
              </div>
              <div style={A.logoEmoji}>🥞</div>
            </div>
          </div>

          {/* Main content — wheel first */}
          <div style={A.body}>

            {/* Headline above wheel */}
            <div style={A.topText}>
              <div style={A.headline}>Spin for <span style={A.headlineAccent}>FREE FOOD</span> &amp; Discounts!</div>
              <div style={A.tagline}>One free spin per visit · Instant prize · No purchase required</div>
            </div>

            {/* Wheel — center stage */}
            <div style={A.wheelCenter}>
              <div style={A.wheelWrap}>
                <div style={A.pointerWrap}><Pointer size={44}/></div>
                <SpinWheel spinning={false} targetIndex={0} onDone={()=>{}} idle={true} size={wheelSize}/>
              </div>
            </div>

            {/* Prize chips in a single row below */}
            <div style={A.prizeRow}>
              {[
                {e:"🏷️",l:"10% Off"},
                {e:"✂️", l:"20% Off"},
                {e:"☕", l:"Free Coffee"},
                {e:"🎉",l:"30% Off"},
                {e:"🔥",l:"40% Off"},
                {e:"💥",l:"50% Off"},
                {e:"🥞",l:"Free Crepe"},
              ].map(p=>(
                <div key={p.l} style={A.prizeChip}>
                  <span style={A.prizeEmoji}>{p.e}</span>
                  <div style={A.prizeLabel}>{p.l}</div>
                </div>
              ))}
            </div>

            {/* CTA */}
            <button style={A.cta} onClick={()=>setModal(true)} className="pulse-btn">
              🎰 TAP TO SPIN!
            </button>
            <div style={A.ctaNote}>Free · One spin per visit</div>

          </div>
        </div>
      )}

      {/* ── SPIN ── */}
      {screen===SCREEN.SPIN&&(
        <div style={A.spinScreen} className="fade-in">
          <SparklesBg/>
          <div style={A.spinBrand}>P.S. I Crepe You 🥞</div>
          <div style={A.spinMsg}>{spinning?"✨ Come on lucky spin…":"Get ready!"}</div>
          <div style={A.wheelWrap}>
            <div style={A.pointerWrap}><Pointer size={44}/></div>
            <SpinWheel spinning={spinning} targetIndex={targetIdx} onDone={onSpinDone} idle={false} size={Math.min(wheelSize+40,620)}/>
          </div>
          {spinning&&<div style={A.spinSub} className="breathe">Hold your breath… 🍀</div>}
        </div>
      )}

      {/* ── RESULT ── */}
      {screen===SCREEN.RESULT&&prize&&(
        <div style={A.resultScreen} className="fade-in">
          <SparklesBg/>
          <div style={A.resultInner}>

            {/* Trophy / emoji */}
            <div style={A.resultEmoji} className="bounce-in">{prize.emoji}</div>
            <div style={A.resultYouWon}>🎉 YOU WON</div>
            <div style={{...A.resultPrize, color:tierAccent(prize.tier)}}>{prize.label}!</div>

            {/* Code card */}
            <div style={{...A.codeCard, borderColor:tierAccent(prize.tier)+"88"}}>
              <div style={A.codeCardSheen}/>
              <div style={A.codeTag}>PRIZE CODE</div>
              <div style={A.codeVal}>{code}</div>
              <div style={A.codeDivider}/>
              <div style={A.codeInstr}>Show this screen to your cashier</div>
              <div style={A.codeInstr2}>Tell them: <strong>"{prize.label}"</strong> — Code: <strong>{code}</strong></div>
            </div>

            {/* SMS notice */}
            <div style={A.smsCard}>
              <div style={{fontSize:36}}>💬</div>
              <div>
                <div style={A.smsTitle}>Check your texts!</div>
                <div style={A.smsSub}>We just sent you a link to leave us a Google review. It means the world to our small family business. 💕</div>
              </div>
            </div>

            <button style={A.resetBtn} onClick={reset}>← New Customer</button>
          </div>
        </div>
      )}
    </div>
  );
}

// ─── STYLES ───────────────────────────────────────────────────────────────────
const A = {
  root:        { minHeight:"100vh", minWidth:"100vw", background:"linear-gradient(145deg,#fff0f5 0%,#fce4ec 50%,#f3e5f5 100%)", fontFamily:"Georgia,serif", overflow:"hidden", position:"relative" },
  // HOME
  home:        { position:"relative", display:"flex", flexDirection:"column", minHeight:"100vh" },
  header:      { background:`linear-gradient(135deg,${DPINK},${PINK})`, padding:"20px 40px", boxShadow:`0 4px 24px ${PINK}55`, position:"relative", zIndex:1 },
  headerInner: { display:"flex", justifyContent:"space-between", alignItems:"center", maxWidth:1100, margin:"0 auto", width:"100%" },
  brand:       { color:"#fff", fontWeight:900, fontSize:30, letterSpacing:0.5, textShadow:"0 2px 10px #0003" },
  brandSub:    { color:"rgba(255,255,255,0.75)", fontSize:13, marginTop:4, letterSpacing:0.5 },
  logoEmoji:   { fontSize:48 },
  body:        { flex:1, display:"flex", flexDirection:"column", alignItems:"center", justifyContent:"center", gap:20, padding:"24px 48px 32px", maxWidth:1100, margin:"0 auto", width:"100%", position:"relative", zIndex:1 },
  topText:     { textAlign:"center" },
  headline:    { fontWeight:900, fontSize:48, lineHeight:1.15, color:"#2d0a14", marginBottom:8 },
  headlineAccent:{ color:PINK, textShadow:`0 2px 20px ${PINK}44` },
  tagline:     { color:"#8b4460", fontSize:15, letterSpacing:0.3 },
  wheelCenter: { display:"flex", alignItems:"center", justifyContent:"center" },
  prizeRow:    { display:"flex", gap:10, flexWrap:"wrap", justifyContent:"center" },
  prizeChip:   { display:"flex", flexDirection:"column", alignItems:"center", gap:4, background:"rgba(255,255,255,0.75)", borderRadius:16, padding:"10px 16px", border:`1px solid ${PINK}22`, backdropFilter:"blur(6px)", minWidth:72 },
  prizeEmoji:  { fontSize:22 },
  prizeLabel:  { fontWeight:700, fontSize:12, color:"#2d0a14", textAlign:"center" },
  cta:         { padding:"22px 80px", background:`linear-gradient(135deg,${PINK},${DPINK})`, color:"#fff", fontWeight:900, fontSize:26, border:"none", borderRadius:20, cursor:"pointer", fontFamily:"Georgia,serif", letterSpacing:1.5, boxShadow:`0 8px 32px ${PINK}66` },
  ctaNote:     { color:"#c48ba0", fontSize:11, textAlign:"center" },
  wheelWrap:   { position:"relative", display:"inline-flex", alignItems:"center", justifyContent:"center" },
  pointerWrap: { position:"absolute", top:-28, left:"50%", transform:"translateX(-50%)", zIndex:10 },
  // SPIN
  spinScreen:  { position:"relative", minHeight:"100vh", display:"flex", flexDirection:"column", alignItems:"center", justifyContent:"center", gap:20, padding:32 },
  spinBrand:   { color:PINK, fontWeight:900, fontSize:20, position:"relative", zIndex:1 },
  spinMsg:     { fontWeight:700, fontSize:26, color:"#2d0a14", position:"relative", zIndex:1 },
  spinSub:     { color:PINK, fontWeight:700, fontSize:18, marginTop:8, position:"relative", zIndex:1 },
  // RESULT
  resultScreen:{ position:"relative", minHeight:"100vh", display:"flex", alignItems:"center", justifyContent:"center", padding:"32px 24px" },
  resultInner: { position:"relative", zIndex:1, display:"flex", flexDirection:"column", alignItems:"center", maxWidth:560, width:"100%", textAlign:"center", gap:16 },
  resultEmoji: { fontSize:100, lineHeight:1, filter:"drop-shadow(0 8px 20px rgba(232,69,122,0.4))" },
  resultYouWon:{ fontSize:13, letterSpacing:4, color:"#c48ba0", fontWeight:700 },
  resultPrize: { fontWeight:900, fontSize:52, lineHeight:1.1 },
  codeCard:    { width:"100%", background:"#fff", borderRadius:24, border:"2px solid", boxShadow:"0 12px 48px #e8457a1a", padding:"28px 32px", position:"relative", overflow:"hidden" },
  codeCardSheen:{ position:"absolute", top:0, left:"-100%", width:"60%", height:"100%", background:"linear-gradient(105deg,transparent,rgba(255,255,255,0.5),transparent)", animation:"sheen 3s ease-in-out infinite", pointerEvents:"none" },
  codeTag:     { fontSize:10, letterSpacing:4, color:PINK, fontWeight:700, marginBottom:10 },
  codeVal:     { fontFamily:"monospace", fontWeight:900, fontSize:40, letterSpacing:8, color:"#2d0a14", marginBottom:16 },
  codeDivider: { height:1, background:`linear-gradient(90deg,transparent,${PINK}44,transparent)`, marginBottom:16 },
  codeInstr:   { fontSize:14, color:"#8b4460", marginBottom:4 },
  codeInstr2:  { fontSize:13, color:"#2d0a14" },
  smsCard:     { width:"100%", display:"flex", gap:16, alignItems:"flex-start", background:"rgba(255,255,255,0.85)", borderRadius:20, padding:"20px 24px", border:`1px solid ${PINK}22`, backdropFilter:"blur(8px)" },
  smsTitle:    { fontWeight:800, fontSize:17, color:"#2d0a14", marginBottom:4 },
  smsSub:      { fontSize:13, color:"#8b4460", lineHeight:1.6 },
  resetBtn:    { background:"none", border:`1px solid ${PINK}44`, color:"#c48ba0", padding:"12px 32px", borderRadius:14, cursor:"pointer", fontFamily:"Georgia,serif", fontSize:14 },
};

const MO = {
  overlay: { position:"fixed", inset:0, background:"rgba(45,10,20,0.6)", zIndex:200, display:"flex", alignItems:"center", justifyContent:"center", padding:32, backdropFilter:"blur(6px)" },
  modal:   { background:"#fff", borderRadius:32, padding:"44px 40px 36px", width:"100%", maxWidth:500, textAlign:"center", position:"relative", boxShadow:"0 32px 80px #0005" },
  close:   { position:"absolute", top:16, right:20, background:"none", border:"none", fontSize:22, color:"#c48ba0", cursor:"pointer" },
  icon:    { fontSize:60, marginBottom:12 },
  title:   { fontWeight:900, fontSize:28, color:"#2d0a14", marginBottom:10 },
  sub:     { color:"#8b4460", fontSize:15, lineHeight:1.65, marginBottom:24 },
  input:   { width:"100%", padding:"18px", borderRadius:18, border:"2px solid #f8bbd0", fontSize:26, color:"#2d0a14", outline:"none", fontFamily:"Georgia,serif", textAlign:"center", letterSpacing:4, marginBottom:14, boxSizing:"border-box", background:"#fff9fb" },
  inputErr:{ borderColor:PINK, boxShadow:`0 0 0 4px ${PINK}22` },
  row:     { display:"flex", alignItems:"center", gap:14, background:LIGHT, borderRadius:16, padding:"16px 18px", cursor:"pointer", marginBottom:12, border:"2px solid", transition:"border-color 0.2s" },
  cb:      { width:28, height:28, borderRadius:8, border:`2px solid ${PINK}`, background:"#fff", display:"flex", alignItems:"center", justifyContent:"center", flexShrink:0 },
  cbOn:    { background:PINK, borderColor:DPINK },
  cbLabel: { fontWeight:700, fontSize:15, color:"#2d0a14", marginBottom:3 },
  cbSub:   { fontSize:12, color:"#8b4460" },
  err:     { color:PINK, fontSize:13, marginBottom:8 },
  legal:   { fontSize:11, color:"#c48ba0", lineHeight:1.6, marginBottom:14, textAlign:"left" },
  btn:     { width:"100%", padding:"18px", background:`linear-gradient(135deg,${PINK},${DPINK})`, color:"#fff", fontWeight:900, fontSize:21, border:"none", borderRadius:20, cursor:"pointer", fontFamily:"Georgia,serif", boxShadow:`0 6px 24px ${PINK}55`, letterSpacing:0.5 },
};

const CSS = `
  @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&display=swap');
  * { box-sizing:border-box; margin:0; padding:0; }
  .fade-in   { animation: fadeUp 0.5s ease both; }
  .modal-in  { animation: modalPop 0.4s cubic-bezier(0.34,1.56,0.64,1) both; }
  .bounce-in { animation: bounceIn 0.6s cubic-bezier(0.34,1.56,0.64,1) both; }
  .breathe   { animation: breathe 1.8s ease-in-out infinite; }
  .pulse-btn { animation: pulsePink 2.2s ease-in-out infinite; }
  @keyframes fadeUp    { from{opacity:0;transform:translateY(20px)} to{opacity:1;transform:none} }
  @keyframes modalPop  { from{opacity:0;transform:scale(0.85) translateY(24px)} to{opacity:1;transform:none} }
  @keyframes bounceIn  { from{opacity:0;transform:scale(0.4) rotate(-10deg)} 60%{transform:scale(1.15) rotate(5deg)} to{opacity:1;transform:none} }
  @keyframes breathe   { 0%,100%{opacity:0.7;transform:scale(1)} 50%{opacity:1;transform:scale(1.04)} }
  @keyframes pulsePink { 0%,100%{box-shadow:0 8px 32px rgba(232,69,122,0.5)} 50%{box-shadow:0 8px 48px rgba(232,69,122,0.85),0 0 0 10px rgba(232,69,122,0.12)} }
  @keyframes sheen     { 0%{left:-100%} 60%,100%{left:150%} }
  @keyframes sparkle   { 0%,100%{opacity:0.1;transform:scale(0.7)} 50%{opacity:0.4;transform:scale(1)} }
  button:not(:disabled):active { transform:scale(0.97) !important; }
`;
