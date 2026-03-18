import { useState, useRef, useEffect } from "react";

const B = { primary:"#1B7E9F", dark:"#0F5570", light:"#E8F4F8", mid:"#4BA3C3" };

// type:"qty"   → comptage par unité (capsules, sachets, etc.)
// type:"level" → niveau de remplissage (plein/3/4/1/2/1/4/vide) — alerte à ≤ 1/4
const INIT_PRODUITS = [
  { id:"capsules",     label:"Capsules café",          icon:"☕", cat:"Bienvenue",    type:"qty",   perV:1, perS:0, min:6,  unite:"unité",   orderUrl:"https://www.amazon.fr/s?k=capsules+nespresso",      orderLabel:"Amazon" },
  { id:"the",          label:"Sachets de thé",          icon:"🍵", cat:"Bienvenue",    type:"qty",   perV:1, perS:0, min:6,  unite:"sachet",  orderUrl:"https://www.amazon.fr/s?k=sachets+the+vrac",        orderLabel:"Amazon" },
  { id:"speculoos",    label:"Spéculoos",               icon:"🍪", cat:"Bienvenue",    type:"qty",   perV:1, perS:0, min:8,  unite:"biscuit", orderUrl:"https://www.amazon.fr/s?k=speculoos+vrac",          orderLabel:"Amazon" },
  { id:"gel_douche",   label:"Gel douche",              icon:"🚿", cat:"Salle de bain",type:"level",                         unite:"flacon",  orderUrl:"https://www.amazon.fr/s?k=gel+douche+hotel",        orderLabel:"Amazon" },
  { id:"shampoing",    label:"Shampoing",               icon:"💆", cat:"Salle de bain",type:"level",                         unite:"flacon",  orderUrl:"https://www.amazon.fr/s?k=shampoing+miniature",     orderLabel:"Amazon" },
  { id:"papier_wc",    label:"Papier toilette",         icon:"🧻", cat:"Salle de bain",type:"qty",   perV:0, perS:2, min:4,  unite:"rouleau", orderUrl:"https://www.amazon.fr/s?k=papier+toilette",         orderLabel:"Amazon" },
  { id:"dosettes",     label:"Dosettes lave-vaisselle", icon:"🧼", cat:"Cuisine",      type:"qty",   perV:0, perS:1, min:4,  unite:"dosette", orderUrl:"https://www.amazon.fr/s?k=dosettes+lave+vaisselle", orderLabel:"Amazon" },
  { id:"liquide_vais", label:"Liquide vaisselle",       icon:"🫧", cat:"Cuisine",      type:"level",                         unite:"flacon",  orderUrl:"https://www.amazon.fr/s?k=liquide+vaisselle",       orderLabel:"Amazon" },
  { id:"eponge",       label:"Éponge",                  icon:"🧽", cat:"Entretien",    type:"qty",   perV:0, perS:1, min:2,  unite:"unité",   orderUrl:"https://www.amazon.fr/s?k=eponge+cuisine",          orderLabel:"Amazon" },
  { id:"sacs_poubelle",label:"Sacs poubelle",           icon:"🗑", cat:"Entretien",    type:"qty",   perV:0, perS:2, min:6,  unite:"sac",     orderUrl:"https://www.amazon.fr/s?k=sacs+poubelle",           orderLabel:"Amazon" },
  { id:"nettoyant_sol",label:"Nettoyant sol",           icon:"🧹", cat:"Entretien",    type:"level",                         unite:"flacon",  orderUrl:"https://www.amazon.fr/s?k=nettoyant+sol+multi+usage", orderLabel:"Amazon" },
  { id:"nettoyant_vit",label:"Nettoyant vitres",        icon:"🪟", cat:"Entretien",    type:"level",                         unite:"flacon",  orderUrl:"https://www.amazon.fr/s?k=nettoyant+vitres",        orderLabel:"Amazon" },
  { id:"detartrant",   label:"Détartrant toilettes",    icon:"🚽", cat:"Entretien",    type:"level",                         unite:"flacon",  orderUrl:"https://www.amazon.fr/s?k=detartrant+toilettes",    orderLabel:"Amazon" },
];

const INIT_STOCKS = {
  "villa-azur":    {capsules:{qty:12,at:"03-08",by:"p1"},the:{qty:8,at:"03-08",by:"p1"},speculoos:{qty:10,at:"03-08",by:"p1"},gel_douche:{level:"3/4",at:"03-08",by:"p1"},shampoing:{level:"full",at:"03-08",by:"p1"},papier_wc:{qty:6,at:"03-08",by:"p1"},dosettes:{qty:5,at:"03-08",by:"p1"},liquide_vais:{level:"1/2",at:"03-08",by:"p1"},eponge:{qty:1,at:"03-08",by:"p1"},sacs_poubelle:{qty:4,at:"03-08",by:"p1"},nettoyant_sol:{level:"1/4",at:"03-08",by:"p1"},nettoyant_vit:{level:"full",at:"03-08",by:"p1"},detartrant:{level:"3/4",at:"03-08",by:"p1"}},
  "apt-lumiere":   {capsules:{qty:4,at:"03-01",by:"p2"},the:{qty:3,at:"03-01",by:"p2"},speculoos:{qty:5,at:"03-01",by:"p2"},gel_douche:{level:"1/2",at:"03-01",by:"p2"},shampoing:{level:"1/4",at:"03-01",by:"p2"},papier_wc:{qty:2,at:"03-01",by:"p2"},dosettes:{qty:2,at:"03-01",by:"p2"},liquide_vais:{level:"full",at:"03-01",by:"p2"},eponge:{qty:0,at:"03-01",by:"p2"},sacs_poubelle:{qty:3,at:"03-01",by:"p2"},nettoyant_sol:{level:"full",at:"03-01",by:"p2"},nettoyant_vit:{level:"3/4",at:"03-01",by:"p2"},detartrant:{level:"1/2",at:"03-01",by:"p2"}},
  "chalet-mont":   {capsules:{qty:18,at:"02-20",by:"p3"},the:{qty:12,at:"02-20",by:"p3"},speculoos:{qty:16,at:"02-20",by:"p3"},gel_douche:{level:"full",at:"02-20",by:"p3"},shampoing:{level:"full",at:"02-20",by:"p3"},papier_wc:{qty:8,at:"02-20",by:"p3"},dosettes:{qty:6,at:"02-20",by:"p3"},liquide_vais:{level:"3/4",at:"02-20",by:"p3"},eponge:{qty:3,at:"02-20",by:"p3"},sacs_poubelle:{qty:8,at:"02-20",by:"p3"},nettoyant_sol:{level:"full",at:"02-20",by:"p3"},nettoyant_vit:{level:"full",at:"02-20",by:"p3"},detartrant:{level:"full",at:"02-20",by:"p3"}},
  "mas-provence":  {capsules:{qty:5,at:"02-25",by:"p1"},the:{qty:4,at:"02-25",by:"p1"},speculoos:{qty:3,at:"02-25",by:"p1"},gel_douche:{level:"1/4",at:"02-25",by:"p1"},shampoing:{level:"1/2",at:"02-25",by:"p1"},papier_wc:{qty:3,at:"02-25",by:"p1"},dosettes:{qty:3,at:"02-25",by:"p1"},liquide_vais:{level:"1/4",at:"02-25",by:"p1"},eponge:{qty:2,at:"02-25",by:"p1"},sacs_poubelle:{qty:5,at:"02-25",by:"p1"},nettoyant_sol:{level:"1/2",at:"02-25",by:"p1"},nettoyant_vit:{level:"1/4",at:"02-25",by:"p1"},detartrant:{level:"1/4",at:"02-25",by:"p1"}},
  "loft-bordeaux": {capsules:{qty:8,at:"03-08",by:"p2"},the:{qty:6,at:"03-08",by:"p2"},speculoos:{qty:9,at:"03-08",by:"p2"},gel_douche:{level:"3/4",at:"03-08",by:"p2"},shampoing:{level:"3/4",at:"03-08",by:"p2"},papier_wc:{qty:5,at:"03-08",by:"p2"},dosettes:{qty:4,at:"03-08",by:"p2"},liquide_vais:{level:"full",at:"03-08",by:"p2"},eponge:{qty:2,at:"03-08",by:"p2"},sacs_poubelle:{qty:6,at:"03-08",by:"p2"},nettoyant_sol:{level:"3/4",at:"03-08",by:"p2"},nettoyant_vit:{level:"1/2",at:"03-08",by:"p2"},detartrant:{level:"full",at:"03-08",by:"p2"}},
};

const INIT_PROPS = [
  {id:"villa-azur",    name:"Villa Azur",       address:"12 rue de la Mer, Nice",            access:"Boîte à clé : code 4521 — portail gauche",    duration:180, lodgifyKey:"lod_vjg8X"},
  {id:"apt-lumiere",   name:"Apt. Lumière",      address:"8 bd Haussmann, Paris 9e",          access:"Clé chez gardienne Mme Rousseau — bât. B",    duration:120, lodgifyKey:"lod_aHq3"},
  {id:"chalet-mont",   name:"Chalet Mont-Blanc", address:"3 chemin des Cimes, Chamonix",      access:"Clé sous pot de fleurs escalier droite",      duration:240, lodgifyKey:"lod_cKw5"},
  {id:"mas-provence",  name:"Mas Provence",      address:"Rte des Oliviers, Aix-en-Provence", access:"Boîte à clé : code 8873 — entrée principale", duration:210, lodgifyKey:"lod_mBr6"},
  {id:"loft-bordeaux", name:"Loft Bordeaux",     address:"15 quai des Chartrons, Bordeaux",   access:"Digicode entrée : 3B492 — apt 12 3e étage",  duration:90,  lodgifyKey:"lod_lDs4"},
];

const INIT_PRESTATAIRES = [
  {id:"p1",name:"Jean-Marc Dupont",role:"Ménage & Entretien",  pin:"1111",email:"jm.dupont@email.fr",  phone:"06 12 34 56 78",properties:["villa-azur","mas-provence"]},
  {id:"p2",name:"Sophie Lemaire",  role:"Nettoyage & Linge",   pin:"2222",email:"s.lemaire@email.fr",  phone:"06 23 45 67 89",properties:["apt-lumiere","loft-bordeaux"]},
  {id:"p3",name:"Carlos Fernandez",role:"Maintenance générale",pin:"3333",email:"c.fernandez@email.fr",phone:"06 34 56 78 90",properties:["chalet-mont","villa-azur"]},
];

const INIT_MESSAGES = [
  {id:"msg1",date:"2026-03-09",subject:"Rappel : check-in à 15h00",body:"Bonjour,\n\nLe check-in est fixé à 15h00. Les logements doivent être prêts au plus tard à 14h45.\n\nMerci.",targets:"all",pinned:true, type:"info"},
  {id:"msg2",date:"2026-03-07",subject:"Nouveau produit : Ecover",body:"Nous passons à la gamme Ecover (éco-responsable). Produits en place depuis le 10 mars.",targets:"all",pinned:false,type:"info"},
  {id:"msg3",date:"2026-03-05",subject:"Photos PPL : qualité",body:"Photos floues sur plusieurs rapports récents. Allumez toutes les lumières avant de photographier. Minimum 4 photos générales.",targets:"all",pinned:false,type:"warning"},
];

const INIT_PPLS = [
  {id:"m1",property:"villa-azur",   date:"2026-03-10",timeSlot:"11:30",prestataire:"p1",status:"acceptée",  guest:"Famille Martin",guestCount:3,nextCheckin:"14:00",source:"lodgify",candidates:[],vigilancePoints:["Miroir salle de bain — traces fréquentes","Serviettes mal pliées","Kit bienvenue incomplet"]},
  {id:"m2",property:"apt-lumiere",  date:"2026-03-10",timeSlot:"10:30",prestataire:"p2",status:"acceptée",  guest:"M. Chen",       guestCount:1,nextCheckin:"16:00",source:"lodgify",candidates:[],vigilancePoints:["Poussière sous le lit","Fenêtre salon à vérifier"]},
  {id:"m3",property:"chalet-mont",  date:"2026-03-12",timeSlot:"10:00",prestataire:"",  status:"disponible",guest:"Famille Dubois",guestCount:4,nextCheckin:"15:00",source:"manuel", candidates:["p3"],vigilancePoints:[]},
  {id:"m4",property:"mas-provence", date:"2026-03-14",timeSlot:"11:00",prestataire:"",  status:"disponible",guest:"Mme Leroy",     guestCount:2,nextCheckin:"17:00",source:"lodgify",candidates:[],vigilancePoints:[]},
  {id:"m5",property:"loft-bordeaux",date:"2026-03-08",timeSlot:"10:00",prestataire:"p2",status:"terminée",  guest:"M. Park",       guestCount:2,nextCheckin:"14:00",source:"lodgify",candidates:[],vigilancePoints:[]},
  {id:"m6",property:"chalet-mont",  date:"2026-03-19",timeSlot:"09:00",prestataire:"",  status:"disponible",guest:"M. Weber",       guestCount:2,nextCheckin:"15:00",source:"lodgify",candidates:[],vigilancePoints:["Score 92/100 — maintenir"]},
];

const UPCOMING = {
  "villa-azur":    [{date:"2026-03-10",guests:3},{date:"2026-03-14",guests:2},{date:"2026-03-17",guests:4}],
  "apt-lumiere":   [{date:"2026-03-10",guests:1},{date:"2026-03-18",guests:2}],
  "chalet-mont":   [{date:"2026-03-12",guests:4},{date:"2026-03-19",guests:2}],
  "mas-provence":  [{date:"2026-03-14",guests:2},{date:"2026-03-21",guests:3}],
  "loft-bordeaux": [{date:"2026-03-15",guests:1}],
};

const PPL_HIST = {
  "villa-azur":   [{date:"2026-02-28",by:"p1",score:72,issues:["Serviettes mal pliées","Miroir traces"],goods:["Cuisine impeccable"]},{date:"2026-02-14",by:"p1",score:81,issues:["Literie mal tendue"],goods:["Salle de bain parfaite"]}],
  "apt-lumiere":  [{date:"2026-03-01",by:"p2",score:85,issues:["Poussière sous lit"],goods:["Salle de bain excellente"]}],
  "chalet-mont":  [{date:"2026-02-20",by:"p3",score:92,issues:[],goods:["Mission parfaite"]}],
  "mas-provence": [{date:"2026-02-25",by:"p1",score:61,issues:["Tache canapé non signalée"],goods:["Cuisine OK"]}],
  "loft-bordeaux":[{date:"2026-03-08",by:"p2",score:96,issues:[],goods:["Excellent"]}],
};

const TODAY = "2026-03-10";
const WDAYS = ["Lun","Mar","Mer","Jeu","Ven","Sam","Dim"];
const CAT_C = {Bienvenue:{bg:"#F0F9FF",bd:"#93C5FD"},"Salle de bain":{bg:"#FDF4FF",bd:"#D8B4FE"},Cuisine:{bg:"#FFFBEB",bd:"#FCD34D"},Entretien:{bg:"#F0FDF4",bd:"#6EE7B7"}};

function weekDates(anchor) {
  const d=new Date(anchor),day=d.getDay(),diff=day===0?-6:1-day;
  const mon=new Date(d);mon.setDate(d.getDate()+diff);
  return Array.from({length:7},(_,i)=>{const x=new Date(mon);x.setDate(mon.getDate()+i);return x.toISOString().split("T")[0];});
}

async function callClaude(sys,msg) {
  const r=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:600,system:sys,messages:[{role:"user",content:msg}]})});
  const d=await r.json();
  return d.content?.find(b=>b.type==="text")?.text||"";
}

async function analyzeImg(file,sys,msg) {
  const b64=await new Promise((res,rej)=>{const r=new FileReader();r.onload=()=>res(r.result.split(",")[1]);r.onerror=rej;r.readAsDataURL(file);});
  const r=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:600,system:sys,messages:[{role:"user",content:[{type:"image",source:{type:"base64",media_type:file.type||"image/jpeg",data:b64}},{type:"text",text:msg}]}]})});
  const d=await r.json();
  return (d.content?.find(b=>b.type==="text")?.text||"{}").replace(/```json|```/g,"").trim();
}

// ─── SHARED UI ────────────────────────────────────────────────────────────────
const Logo = ({size="md"}) => {
  const fs=size==="lg"?26:size==="sm"?14:18, h=size==="lg"?22:size==="sm"?12:16;
  return <div style={{display:"flex",alignItems:"center",userSelect:"none"}}>
    {"LOMMAX".split("").map((l,i)=><div key={i} style={{display:"flex",alignItems:"center"}}>
      <span style={{fontSize:fs,fontWeight:300,letterSpacing:"0.04em",color:B.primary,fontFamily:"'Helvetica Neue',Arial,sans-serif",lineHeight:1}}>{l}</span>
      {i<5&&<div style={{width:1,height:h,background:B.primary,margin:`0 ${size==="sm"?3:5}px`,opacity:0.55}}/>}
    </div>)}
  </div>;
};

const Bdg = ({color,bg,children}) => <span style={{fontSize:10,fontWeight:600,padding:"3px 8px",borderRadius:18,background:bg||color+"18",color,textTransform:"uppercase",letterSpacing:"0.05em",whiteSpace:"nowrap"}}>{children}</span>;

const Btn = ({onClick,disabled,v="primary",children,sx={}}) => {
  const S={primary:{background:B.primary,color:"#fff",border:"none"},outline:{background:"#fff",border:"1px solid #D1DAE3",color:"#374151"},green:{background:"#ECFDF5",border:"1px solid #6EE7B7",color:"#065F46"},danger:{background:"#FFF1F1",border:"1px solid #FCA5A5",color:"#991B1B"},ghost:{background:"transparent",border:"none",color:B.primary}};
  return <button onClick={onClick} disabled={disabled} style={{borderRadius:9,padding:"10px 18px",fontSize:13,fontWeight:600,cursor:disabled?"not-allowed":"pointer",display:"flex",alignItems:"center",justifyContent:"center",gap:6,opacity:disabled?0.5:1,...S[v],...sx}}>{children}</button>;
};

const Av = ({name,size=36}) => <div style={{width:size,height:size,borderRadius:"50%",background:B.light,border:`2px solid ${B.primary}33`,display:"flex",alignItems:"center",justifyContent:"center",fontSize:size*0.33,fontWeight:700,color:B.primary,flexShrink:0}}>{name.split(" ").map(n=>n[0]).join("").slice(0,2)}</div>;

const SD = ({score}) => {const c=score>=80?"#10B981":score>=60?"#F59E0B":"#EF4444";return <div style={{width:30,height:30,borderRadius:"50%",background:"#fff",border:`3px solid ${c}`,display:"flex",alignItems:"center",justifyContent:"center",fontSize:10,fontWeight:800,color:c,flexShrink:0}}>{score}</div>;};

const Modal = ({onClose,children,w="min(600px,95vw)"}) => (
  <div style={{position:"fixed",inset:0,background:"#00000066",zIndex:300,display:"flex",alignItems:"center",justifyContent:"center"}} onClick={onClose}>
    <div style={{background:"#fff",borderRadius:16,padding:24,width:w,maxHeight:"90vh",overflow:"auto",boxShadow:"0 20px 60px #00000030"}} onClick={e=>e.stopPropagation()}>{children}</div>
  </div>
);

const Fld = ({label,children}) => <div style={{marginBottom:11}}><div style={{fontSize:11,fontWeight:600,color:"#6B7280",textTransform:"uppercase",letterSpacing:"0.07em",marginBottom:4}}>{label}</div>{children}</div>;
const Inp = ({value,onChange,placeholder,type="text",sx={}}) => <input type={type} value={value} onChange={e=>onChange(e.target.value)} placeholder={placeholder} style={{width:"100%",background:"#F9FAFB",border:"1px solid #D1DAE3",borderRadius:9,padding:"9px 12px",fontSize:13,color:"#111827",outline:"none",boxSizing:"border-box",...sx}}/>;
const Sel = ({value,onChange,children}) => <select value={value} onChange={e=>onChange(e.target.value)} style={{width:"100%",background:"#F9FAFB",border:"1px solid #D1DAE3",borderRadius:9,padding:"9px 12px",fontSize:13,color:"#111827",outline:"none",boxSizing:"border-box"}}>{children}</select>;

const UpZone = ({onFiles,label="Ajouter des photos",multi=true}) => {
  const ref=useRef(null);
  return <div onClick={()=>ref.current?.click()} style={{border:`2px dashed ${B.mid}`,borderRadius:10,padding:"14px",textAlign:"center",cursor:"pointer",background:B.light}}>
    <div style={{fontSize:16,marginBottom:2}}>🖼</div>
    <div style={{fontSize:13,fontWeight:600,color:B.dark}}>{label}</div>
    <div style={{fontSize:11,color:"#9CA3AF"}}>Glissez ou cliquez</div>
    <input ref={ref} type="file" accept="image/*" multiple={multi} style={{display:"none"}} onChange={e=>onFiles(e.target.files)}/>
  </div>;
};

const PhotoRow = ({photos,onRemove}) => photos.length===0?null:(
  <div style={{display:"flex",gap:7,flexWrap:"wrap",marginTop:8}}>
    {photos.map((p,i)=><div key={i} style={{position:"relative",width:80,height:60,borderRadius:8,overflow:"hidden",background:"#f0f0f0",flexShrink:0}}>
      <img src={p.preview||p} alt="" style={{width:"100%",height:"100%",objectFit:"cover"}}/>
      {onRemove&&<button onClick={()=>onRemove(i)} style={{position:"absolute",top:2,right:2,width:18,height:18,borderRadius:"50%",background:"#000000aa",border:"none",color:"#fff",cursor:"pointer",fontSize:10,display:"flex",alignItems:"center",justifyContent:"center"}}>✕</button>}
    </div>)}
  </div>
);

const card = {background:"#fff",border:"1px solid #E5EDF2",borderRadius:12,padding:"16px 20px",marginBottom:12,boxShadow:"0 1px 3px #00000008"};
const ct = {fontSize:10,fontWeight:600,color:"#9CA3AF",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:8};

// ─── MSG STYLES ────────────────────────────────────────────────────────────────
const MST = {info:{icon:"ℹ",bg:"#EFF6FF",bd:"#93C5FD",color:"#1E40AF"},warning:{icon:"⚠",bg:"#FFFBEB",bd:"#FCD34D",color:"#92400E"},success:{icon:"✅",bg:"#ECFDF5",bd:"#6EE7B7",color:"#065F46"}};

const AnnonceCard = ({msg,isNew=false}) => {
  const s=MST[msg.type]||MST.info;
  return <div style={{background:isNew?"#F0F9FF":"#fff",border:`1px solid ${isNew?s.bd:"#E5EDF2"}`,borderLeft:`4px solid ${s.bd}`,borderRadius:12,padding:"14px 16px",marginBottom:10,position:"relative"}}>
    {isNew&&<div style={{position:"absolute",top:12,right:12,width:8,height:8,borderRadius:"50%",background:"#EF4444"}}/>}
    {msg.pinned&&<div style={{fontSize:10,color:"#9CA3AF",marginBottom:3}}>📌 Épinglé</div>}
    <div style={{display:"flex",gap:8,alignItems:"flex-start",marginBottom:6}}>
      <span style={{fontSize:16}}>{s.icon}</span>
      <div><div style={{fontWeight:700,fontSize:14,color:"#111827"}}>{msg.subject}</div><div style={{fontSize:11,color:"#9CA3AF"}}>LOMMAX · {msg.date}</div></div>
    </div>
    <div style={{fontSize:13,color:"#374151",lineHeight:1.7,whiteSpace:"pre-wrap",background:"#F9FAFB",borderRadius:8,padding:"10px 13px"}}>{msg.body}</div>
  </div>;
};

// ─── LOGIN ────────────────────────────────────────────────────────────────────
function LoginScreen({onLogin,prestataires}) {
  const [mode,setMode]=useState(null);
  const [pin,setPin]=useState("");
  const [err,setErr]=useState("");
  const go=()=>{
    if(mode==="manager"){if(pin==="0000"){onLogin("manager",null);}else{setErr("Code incorrect (démo : 0000)");setPin("");}}
    else{const p=prestataires.find(x=>x.pin===pin);if(p){onLogin("prestataire",p);}else{setErr("Code incorrect");setPin("");}}
  };
  return <div style={{minHeight:"100vh",background:`linear-gradient(135deg,${B.light},#fff 60%,#F0F9FF)`,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",fontFamily:"'DM Sans','Segoe UI',sans-serif",padding:20}}>
    <div style={{marginBottom:32,textAlign:"center"}}><Logo size="lg"/><div style={{fontSize:11,color:"#9CAAB8",textTransform:"uppercase",letterSpacing:"0.14em",marginTop:7}}>Gestion immobilière</div></div>
    {!mode
      ?<div style={{display:"flex",gap:12,flexWrap:"wrap",justifyContent:"center"}}>
        {[{id:"manager",icon:"⬡",label:"Gestionnaire",sub:"Dashboard complet"},{id:"prestataire",icon:"⌂",label:"Prestataire",sub:"PPL & Planning"}].map(r=>(
          <div key={r.id} onClick={()=>setMode(r.id)} style={{background:"#fff",border:`2px solid ${B.light}`,borderRadius:16,padding:"22px 26px",cursor:"pointer",textAlign:"center",width:170,boxShadow:"0 2px 12px #00000010"}} onMouseEnter={e=>e.currentTarget.style.borderColor=B.primary} onMouseLeave={e=>e.currentTarget.style.borderColor=B.light}>
            <div style={{fontSize:28,marginBottom:8}}>{r.icon}</div>
            <div style={{fontWeight:700,fontSize:14,color:"#111827",marginBottom:2}}>{r.label}</div>
            <div style={{fontSize:11,color:"#9CA3AF"}}>{r.sub}</div>
          </div>
        ))}
      </div>
      :<div style={{background:"#fff",borderRadius:16,padding:28,width:"min(290px,90vw)",boxShadow:"0 4px 24px #00000012",border:"1px solid #E5EDF2"}}>
        <div style={{textAlign:"center",marginBottom:20}}><div style={{fontSize:24,marginBottom:5}}>{mode==="manager"?"⬡":"⌂"}</div><div style={{fontWeight:700,fontSize:14}}>{mode==="manager"?"Gestionnaire":"Prestataire"}</div></div>
        <input type="password" value={pin} onChange={e=>{setPin(e.target.value);setErr("");}} onKeyDown={e=>e.key==="Enter"&&go()} placeholder="● ● ● ●" style={{width:"100%",background:"#F9FAFB",border:"1px solid #D1DAE3",borderRadius:9,padding:"10px 14px",fontSize:20,textAlign:"center",outline:"none",letterSpacing:"0.4em",marginBottom:7,boxSizing:"border-box"}}/>
        {err&&<div style={{fontSize:12,color:"#EF4444",textAlign:"center",marginBottom:7}}>{err}</div>}
        <Btn onClick={go} sx={{width:"100%"}}>Connexion</Btn>
        <button onClick={()=>{setMode(null);setPin("");setErr("");}} style={{width:"100%",background:"none",border:"none",color:"#9CA3AF",fontSize:12,cursor:"pointer",marginTop:9}}>← Retour</button>
      </div>}
  </div>;
}

// ─── MANAGER STOCKS TAB ──────────────────────────────────────────────────────
function StocksTab({produits,setProduits,stocks,setStocks,properties,stockAlerts}) {
  const [view,setView]=useState("grid");
  const [filterCat,setFilterCat]=useState("Tous");
  const [addModal,setAddModal]=useState(false);
  const [editProd,setEditProd]=useState(null);
  const [editQty,setEditQty]=useState(null);
  const [editQtyVal,setEditQtyVal]=useState(0);
  const [newProd,setNewProd]=useState({label:"",icon:"📦",cat:"Entretien",perV:0,perS:0,min:2,unite:"unité",orderUrl:"",orderLabel:""});

  const cats=[...new Set(produits.map(p=>p.cat))];
  const CATS=["Tous",...cats];
  const filtered=filterCat==="Tous"?produits:produits.filter(p=>p.cat===filterCat);
  const getS=(pId,prId)=>stocks[pId]?.[prId]||{qty:0,level:"full",at:null};
  const LEVELS=["full","3/4","1/2","1/4","empty"];
  const levelLabel={"full":"Plein","3/4":"3/4","1/2":"1/2","1/4":"1/4 ⚠","empty":"Vide ⚠"};
  const levelAlert=l=>l==="1/4"||l==="empty";
  const levelFill={"full":100,"3/4":75,"1/2":50,"1/4":25,"empty":0};
  const isAlert=(pId,prId)=>{
    const s=getS(pId,prId);
    const p=produits.find(x=>x.id===prId);
    if(!p)return false;
    return p.type==="level" ? levelAlert(s.level||"full") : s.qty<p.min;
  };

  // Visual fill bar for level products
  const FillBar=({level,small=false})=>{
    const fill=levelFill[level||"full"]??100;
    const alert=levelAlert(level);
    const color=alert?"#EF4444":fill>=75?"#10B981":"#F59E0B";
    return <div style={{display:"flex",flexDirection:"column",alignItems:"center",gap:2}}>
      <div style={{width:small?22:28,height:small?36:44,borderRadius:4,border:`2px solid ${alert?"#FCA5A5":"#D1D5DB"}`,background:"#F9FAFB",overflow:"hidden",position:"relative",flexShrink:0}}>
        <div style={{position:"absolute",bottom:0,left:0,right:0,height:`${fill}%`,background:color,transition:"height 0.3s",borderRadius:2}}/>
        {/* cap */}
        <div style={{position:"absolute",top:-4,left:"50%",transform:"translateX(-50%)",width:small?10:14,height:4,background:"#D1D5DB",borderRadius:"2px 2px 0 0"}}/>
      </div>
      <span style={{fontSize:small?8:9,fontWeight:700,color,whiteSpace:"nowrap"}}>{levelLabel[level||"full"]}</span>
    </div>;
  };

  return <div>
    <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:20}}>
      <div>
        <div style={{fontSize:20,fontWeight:700,color:"#111827",marginBottom:2}}>Gestion des stocks</div>
        <div style={{fontSize:13,color:"#6B7280"}}>{stockAlerts>0?`🔴 ${stockAlerts} alerte${stockAlerts>1?"s":""} sous le seuil minimum`:"✅ Tous les stocks sont OK"}</div>
      </div>
      <div style={{display:"flex",gap:8}}>
        <div style={{display:"flex",background:"#F3F4F6",borderRadius:9,padding:3,gap:2}}>
          {[["grid","⊞ Tableau"],["catalogue","☰ Catalogue"]].map(([vv,l])=>(
            <button key={vv} onClick={()=>setView(vv)} style={{padding:"6px 12px",borderRadius:7,border:"none",background:view===vv?"#fff":"transparent",color:view===vv?B.primary:"#6B7280",fontSize:12,fontWeight:view===vv?700:400,cursor:"pointer"}}>{l}</button>
          ))}
        </div>
        <Btn onClick={()=>setAddModal(true)}>+ Produit</Btn>
      </div>
    </div>

    {stockAlerts>0&&<div style={{display:"flex",gap:8,flexWrap:"wrap",marginBottom:14}}>
      {properties.map(prop=>{
        const alerts=produits.filter(p=>isAlert(prop.id,p.id));
        if(!alerts.length)return null;
        return <div key={prop.id} style={{background:"#FFF1F1",border:"1px solid #FCA5A5",borderRadius:9,padding:"7px 13px",fontSize:12}}>
          <span style={{fontWeight:700,color:"#991B1B"}}>{prop.name}</span>{" "}
          <span style={{color:"#EF4444"}}>{alerts.map(p=>`${p.icon} ${p.label}`).join(" · ")}</span>
        </div>;
      })}
    </div>}

    <div style={{display:"flex",gap:6,marginBottom:12,flexWrap:"wrap"}}>
      {CATS.map(c=><button key={c} onClick={()=>setFilterCat(c)} style={{padding:"5px 11px",borderRadius:8,border:filterCat===c?"none":"1px solid #E5EDF2",background:filterCat===c?B.primary:"#fff",color:filterCat===c?"#fff":"#6B7280",fontSize:12,fontWeight:filterCat===c?600:400,cursor:"pointer"}}>{c}</button>)}
    </div>

    {view==="grid"&&<div style={{background:"#fff",border:"1px solid #E5EDF2",borderRadius:12,overflow:"hidden"}}>
      {cats.filter(cat=>filtered.some(p=>p.cat===cat)).map(cat=>(
        <div key={cat}>
          <div style={{display:"flex",alignItems:"center",background:(CAT_C[cat]||{bg:"#F9FAFB"}).bg,borderTop:`2px solid ${(CAT_C[cat]||{bd:"#E5EDF2"}).bd}`,padding:"6px 14px",gap:0}}>
            <div style={{width:190,fontSize:10,fontWeight:700,color:"#6B7280",textTransform:"uppercase",flexShrink:0}}>{cat}</div>
            {properties.map(p=><div key={p.id} style={{flex:1,textAlign:"center",fontSize:10,fontWeight:700,color:B.dark}}>{p.name.split(" ")[0]}</div>)}
            <div style={{width:90,textAlign:"center",fontSize:10,fontWeight:700,color:"#9CA3AF"}}>Cmd</div>
          </div>
          {filtered.filter(p=>p.cat===cat).map((prod,pi)=>{
            const alertRow=properties.some(prop=>isAlert(prop.id,prod.id));
            return <div key={prod.id} style={{display:"flex",alignItems:"center",borderBottom:"1px solid #F3F4F6",background:pi%2===0?"#fff":"#FAFAFA"}}>
              <div style={{width:190,flexShrink:0,padding:"9px 14px",borderRight:"1px solid #F3F4F6"}}>
                <div style={{display:"flex",alignItems:"center",gap:6}}>
                  <span style={{fontSize:15}}>{prod.icon}</span>
                  <div><div style={{fontWeight:600,color:"#111827",fontSize:12}}>{prod.label}</div><div style={{fontSize:10,color:"#9CA3AF"}}>{prod.type==="level"?"Niveau — alerte à ¼":`Min ${prod.min} ${prod.unite}`}</div></div>
                </div>
              </div>
              {properties.map(prop=>{
                const s=getS(prop.id,prod.id);
                const alert=isAlert(prop.id,prod.id);
                return <div key={prop.id} style={{flex:1,padding:"7px 4px",textAlign:"center"}}>
                  {prod.type==="level"
                    ?<button onClick={()=>{setEditQty({propId:prop.id,prodId:prod.id,propName:prop.name,prodLabel:prod.label,prodIcon:prod.icon,unite:prod.unite,type:"level"});setEditQtyVal(s.level||"full");}} style={{background:"none",border:"none",cursor:"pointer",padding:2}}>
                      <FillBar level={s.level||"full"} small/>
                    </button>
                    :<button onClick={()=>{setEditQty({propId:prop.id,prodId:prod.id,propName:prop.name,prodLabel:prod.label,prodIcon:prod.icon,unite:prod.unite,type:"qty"});setEditQtyVal(s.qty);}} style={{display:"inline-flex",flexDirection:"column",alignItems:"center",gap:1,cursor:"pointer",background:alert?"#FFF1F1":"#F0FDF4",border:`1px solid ${alert?"#FCA5A5":"#A7F3D0"}`,borderRadius:8,padding:"4px 7px",minWidth:44}}>
                      <span style={{fontSize:14,fontWeight:800,color:alert?"#EF4444":"#10B981",lineHeight:1}}>{s.qty}</span>
                      <span style={{fontSize:9,color:alert?"#EF4444":"#10B981",fontWeight:600}}>{alert?"⚠":"✓"}</span>
                    </button>}
                  {s.at&&<div style={{fontSize:9,color:"#D1D5DB",marginTop:1}}>{s.at}</div>}
                </div>;
              })}
              <div style={{width:90,padding:"7px",textAlign:"center"}}>
                {prod.orderUrl?<a href={prod.orderUrl} target="_blank" rel="noreferrer" style={{display:"inline-flex",alignItems:"center",gap:3,padding:"4px 7px",borderRadius:7,background:alertRow?"#FFF1F1":"#F9FAFB",border:`1px solid ${alertRow?"#FCA5A5":"#E5E7EB"}`,color:alertRow?"#EF4444":B.primary,fontSize:10,fontWeight:600,textDecoration:"none"}}>🛒{alertRow?" ⚠":""}</a>:<span style={{color:"#D1D5DB",fontSize:11}}>—</span>}
              </div>
            </div>;
          })}
        </div>
      ))}
      <div style={{fontSize:11,color:"#9CA3AF",padding:"7px 14px",textAlign:"center",borderTop:"1px solid #F3F4F6"}}>Cliquez sur une quantité pour la modifier</div>
    </div>}

    {view==="catalogue"&&<div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
      {cats.filter(cat=>filtered.some(p=>p.cat===cat)).map(cat=>{
        const cc=CAT_C[cat]||{bg:"#F9FAFB",bd:"#E5EDF2"};
        return <div key={cat} style={{background:"#fff",border:`1px solid ${cc.bd}`,borderTop:`3px solid ${cc.bd}`,borderRadius:12,padding:"14px 16px"}}>
          <div style={{fontSize:10,fontWeight:700,color:"#9CA3AF",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:10}}>{cat}</div>
          {filtered.filter(p=>p.cat===cat).map(prod=>(
            <div key={prod.id} style={{display:"flex",alignItems:"center",gap:9,padding:"8px 0",borderBottom:"1px solid #F3F4F6"}}>
              <span style={{fontSize:17}}>{prod.icon}</span>
              <div style={{flex:1}}>
                <div style={{fontWeight:600,fontSize:13,color:"#111827"}}>{prod.label}</div>
                <div style={{fontSize:10,color:"#9CA3AF"}}>{prod.type==="level"?"Multi-usage — alerte à ¼ contenant":`Min ${prod.min} ${prod.unite}${prod.perV>0?` · ${prod.perV}/voy.`:""}${prod.perS>0?` · ${prod.perS}/séjour`:""}`}</div>
                {prod.orderUrl&&<a href={prod.orderUrl} target="_blank" rel="noreferrer" style={{fontSize:10,color:B.primary,textDecoration:"none",fontWeight:500}}>🛒 {prod.orderLabel}</a>}
              </div>
              <button onClick={()=>setEditProd({...prod})} style={{padding:"4px 7px",borderRadius:7,background:B.light,border:`1px solid ${B.mid}`,color:B.dark,fontSize:11,cursor:"pointer"}}>✏</button>
              <button onClick={()=>setProduits(prev=>prev.filter(p=>p.id!==prod.id))} style={{padding:"4px 7px",borderRadius:7,background:"#FFF1F1",border:"1px solid #FCA5A5",color:"#EF4444",fontSize:11,cursor:"pointer"}}>✕</button>
            </div>
          ))}
        </div>;
      })}
    </div>}

    {editQty&&<Modal onClose={()=>setEditQty(null)} w="min(320px,94vw)">
      <div style={{fontWeight:700,fontSize:14,marginBottom:3}}>{editQty.prodIcon} {editQty.prodLabel}</div>
      <div style={{fontSize:12,color:"#9CA3AF",marginBottom:16}}>{editQty.propName}</div>
      {editQty.type==="level"
        ?<div style={{marginBottom:16}}>
          <div style={{fontSize:11,fontWeight:600,color:"#6B7280",textTransform:"uppercase",letterSpacing:"0.07em",marginBottom:12}}>Niveau du contenant</div>
          <div style={{display:"flex",gap:10,justifyContent:"center",alignItems:"flex-end",marginBottom:10}}>
            {LEVELS.map(lv=>{
              const fill=levelFill[lv];
              const color=levelAlert(lv)?"#EF4444":fill>=75?"#10B981":"#F59E0B";
              const selected=editQtyVal===lv;
              return <button key={lv} onClick={()=>setEditQtyVal(lv)} style={{display:"flex",flexDirection:"column",alignItems:"center",gap:5,background:"none",border:"none",cursor:"pointer",padding:4,opacity:selected?1:0.5,transform:selected?"scale(1.15)":"scale(1)",transition:"all 0.15s"}}>
                <div style={{width:30,height:50,borderRadius:5,border:`2px solid ${selected?color:"#D1D5DB"}`,background:"#F9FAFB",overflow:"hidden",position:"relative"}}>
                  <div style={{position:"absolute",bottom:0,left:0,right:0,height:`${fill}%`,background:color,borderRadius:3}}/>
                  <div style={{position:"absolute",top:-4,left:"50%",transform:"translateX(-50%)",width:14,height:4,background:"#D1D5DB",borderRadius:"2px 2px 0 0"}}/>
                </div>
                <span style={{fontSize:9,fontWeight:700,color}}>{levelLabel[lv]}</span>
              </button>;
            })}
          </div>
          <div style={{background:levelAlert(editQtyVal)?"#FFF1F1":"#ECFDF5",border:`1px solid ${levelAlert(editQtyVal)?"#FCA5A5":"#6EE7B7"}`,borderRadius:9,padding:"8px 12px",fontSize:12,textAlign:"center",color:levelAlert(editQtyVal)?"#991B1B":"#065F46",fontWeight:600}}>
            {levelAlert(editQtyVal)?"🔴 Alerte — niveau critique, pensez à recommander":"✅ Niveau suffisant"}
          </div>
        </div>
        :<div style={{display:"flex",alignItems:"center",gap:10,marginBottom:16}}>
          <button onClick={()=>setEditQtyVal(v=>Math.max(0,v-1))} style={{width:34,height:34,borderRadius:9,border:"1px solid #E5EDF2",background:"#F9FAFB",fontSize:16,cursor:"pointer",fontWeight:700}}>−</button>
          <Inp type="number" value={String(editQtyVal)} onChange={v=>setEditQtyVal(Math.max(0,parseInt(v)||0))} sx={{textAlign:"center",fontSize:20,fontWeight:800}}/>
          <button onClick={()=>setEditQtyVal(v=>v+1)} style={{width:34,height:34,borderRadius:9,border:"1px solid #E5EDF2",background:"#F9FAFB",fontSize:16,cursor:"pointer",fontWeight:700}}>+</button>
        </div>}
      <div style={{display:"flex",gap:8}}>
        <Btn v="outline" onClick={()=>setEditQty(null)}>Annuler</Btn>
        <Btn onClick={()=>{
          setStocks(prev=>({...prev,[editQty.propId]:{...(prev[editQty.propId]||{}),[editQty.prodId]:editQty.type==="level"?{level:editQtyVal,at:TODAY.slice(5),by:"gestionnaire"}:{qty:editQtyVal,at:TODAY.slice(5),by:"gestionnaire"}}}));
          setEditQty(null);
        }} sx={{flex:1}}>💾 Enregistrer</Btn>
      </div>
    </Modal>}

    {addModal&&<Modal onClose={()=>setAddModal(false)}>
      <div style={{fontWeight:700,fontSize:15,marginBottom:16}}>+ Nouveau produit</div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12}}>
        <Fld label="Icône"><Inp value={newProd.icon} onChange={v=>setNewProd(p=>({...p,icon:v}))}/></Fld>
        <Fld label="Nom *"><Inp value={newProd.label} onChange={v=>setNewProd(p=>({...p,label:v}))} placeholder="Ex: Capsules café"/></Fld>
        <Fld label="Catégorie"><Sel value={newProd.cat} onChange={v=>setNewProd(p=>({...p,cat:v}))}>{["Bienvenue","Entretien","Salle de bain","Cuisine"].map(c=><option key={c} value={c}>{c}</option>)}</Sel></Fld>
        <Fld label="Unité"><Inp value={newProd.unite} onChange={v=>setNewProd(p=>({...p,unite:v}))} placeholder="unité / flacon..."/></Fld>
        <Fld label="Seuil min"><Inp type="number" value={String(newProd.min)} onChange={v=>setNewProd(p=>({...p,min:parseInt(v)||1}))}/></Fld>
        <Fld label="Qté/voyageur"><Inp type="number" value={String(newProd.perV)} onChange={v=>setNewProd(p=>({...p,perV:parseInt(v)||0}))}/></Fld>
        <Fld label="Qté/séjour"><Inp type="number" value={String(newProd.perS)} onChange={v=>setNewProd(p=>({...p,perS:parseInt(v)||0}))}/></Fld>
        <div style={{gridColumn:"1/-1"}}><Fld label="Lien commande"><Inp value={newProd.orderUrl} onChange={v=>setNewProd(p=>({...p,orderUrl:v}))} placeholder="https://amazon.fr/..."/></Fld></div>
        <div style={{gridColumn:"1/-1"}}><Fld label="Label bouton"><Inp value={newProd.orderLabel} onChange={v=>setNewProd(p=>({...p,orderLabel:v}))} placeholder="Amazon..."/></Fld></div>
      </div>
      <div style={{display:"flex",gap:8}}>
        <Btn v="outline" onClick={()=>setAddModal(false)}>Annuler</Btn>
        <Btn onClick={()=>{setProduits(prev=>[...prev,{...newProd,id:"p"+Date.now()}]);setNewProd({label:"",icon:"📦",cat:"Entretien",perV:0,perS:0,min:2,unite:"unité",orderUrl:"",orderLabel:""});setAddModal(false);}} disabled={!newProd.label.trim()} sx={{flex:1}}>Ajouter</Btn>
      </div>
    </Modal>}

    {editProd&&<Modal onClose={()=>setEditProd(null)}>
      <div style={{fontWeight:700,fontSize:15,marginBottom:16}}>✏ Modifier</div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12}}>
        <Fld label="Icône"><Inp value={editProd.icon} onChange={v=>setEditProd(p=>({...p,icon:v}))}/></Fld>
        <Fld label="Nom"><Inp value={editProd.label} onChange={v=>setEditProd(p=>({...p,label:v}))}/></Fld>
        <Fld label="Catégorie"><Sel value={editProd.cat} onChange={v=>setEditProd(p=>({...p,cat:v}))}>{["Bienvenue","Entretien","Salle de bain","Cuisine"].map(c=><option key={c} value={c}>{c}</option>)}</Sel></Fld>
        <Fld label="Unité"><Inp value={editProd.unite} onChange={v=>setEditProd(p=>({...p,unite:v}))}/></Fld>
        <Fld label="Seuil min"><Inp type="number" value={String(editProd.min)} onChange={v=>setEditProd(p=>({...p,min:parseInt(v)||1}))}/></Fld>
        <Fld label="Qté/voy"><Inp type="number" value={String(editProd.perV)} onChange={v=>setEditProd(p=>({...p,perV:parseInt(v)||0}))}/></Fld>
        <div style={{gridColumn:"1/-1"}}><Fld label="Lien"><Inp value={editProd.orderUrl||""} onChange={v=>setEditProd(p=>({...p,orderUrl:v}))}/></Fld></div>
        <div style={{gridColumn:"1/-1"}}><Fld label="Label"><Inp value={editProd.orderLabel||""} onChange={v=>setEditProd(p=>({...p,orderLabel:v}))}/></Fld></div>
      </div>
      <div style={{display:"flex",gap:8}}>
        <Btn v="outline" onClick={()=>setEditProd(null)}>Annuler</Btn>
        <Btn onClick={()=>{setProduits(prev=>prev.map(p=>p.id===editProd.id?editProd:p));setEditProd(null);}} sx={{flex:1}}>💾 Enregistrer</Btn>
      </div>
    </Modal>}
  </div>;
}

// ─── ANNONCES TAB (GESTIONNAIRE) ─────────────────────────────────────────────
function AnnoncesTab({messages,setMessages,prestataires}) {
  const [form,setForm]=useState({subject:"",body:"",targets:"all",type:"info",pinned:false});
  const [compose,setCompose]=useState(false);
  return <div>
    <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:20}}>
      <div><div style={{fontSize:20,fontWeight:700,color:"#111827",marginBottom:2}}>Annonces</div><div style={{fontSize:13,color:"#6B7280"}}>Messages aux prestataires — anonymat total entre eux</div></div>
      <Btn onClick={()=>setCompose(!compose)}>{compose?"✕ Annuler":"+ Nouvelle annonce"}</Btn>
    </div>
    {compose&&<div style={{...card,border:`1px solid ${B.mid}`,marginBottom:16}}>
      <div style={{fontWeight:700,fontSize:14,color:B.dark,marginBottom:12}}>✍ Rédiger</div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:10}}>
        <Fld label="Objet *"><Inp value={form.subject} onChange={v=>setForm(p=>({...p,subject:v}))} placeholder="Titre..."/></Fld>
        <Fld label="Type"><Sel value={form.type} onChange={v=>setForm(p=>({...p,type:v}))}><option value="info">ℹ Information</option><option value="warning">⚠ Attention</option><option value="success">✅ Bonne nouvelle</option></Sel></Fld>
        <div style={{gridColumn:"1/-1"}}>
          <Fld label="Destinataires">
            <div style={{display:"flex",gap:7,flexWrap:"wrap"}}>
              <button onClick={()=>setForm(p=>({...p,targets:"all"}))} style={{padding:"6px 12px",borderRadius:8,border:form.targets==="all"?"none":"1px solid #E5EDF2",background:form.targets==="all"?B.primary:"#F9FAFB",color:form.targets==="all"?"#fff":"#374151",fontSize:12,cursor:"pointer",fontWeight:form.targets==="all"?600:400}}>Tous</button>
              {prestataires.map(p=>{const sel=Array.isArray(form.targets)&&form.targets.includes(p.id);return <button key={p.id} onClick={()=>{const cur=Array.isArray(form.targets)?form.targets:[];setForm(f=>({...f,targets:sel?cur.filter(x=>x!==p.id):[...cur,p.id]}));}} style={{padding:"6px 12px",borderRadius:8,border:sel?"none":"1px solid #E5EDF2",background:sel?B.primary:"#F9FAFB",color:sel?"#fff":"#374151",fontSize:12,cursor:"pointer",fontWeight:sel?600:400}}>{p.name.split(" ")[0]}</button>;})}
            </div>
            <div style={{fontSize:11,color:"#9CA3AF",marginTop:4}}>Les prestataires ne voient pas les autres destinataires.</div>
          </Fld>
        </div>
        <div style={{gridColumn:"1/-1"}}><Fld label="Message *"><textarea value={form.body} onChange={e=>setForm(p=>({...p,body:e.target.value}))} placeholder="Rédigez votre message..." style={{width:"100%",background:"#F9FAFB",border:"1px solid #D1DAE3",borderRadius:9,padding:"9px 12px",fontSize:13,outline:"none",resize:"vertical",minHeight:100,boxSizing:"border-box",fontFamily:"inherit",lineHeight:1.7}}/></Fld></div>
        <label style={{display:"flex",alignItems:"center",gap:7,cursor:"pointer",fontSize:13,color:"#374151"}}><input type="checkbox" checked={form.pinned} onChange={e=>setForm(p=>({...p,pinned:e.target.checked}))} style={{width:15,height:15,accentColor:B.primary}}/>📌 Épingler</label>
      </div>
      <div style={{display:"flex",justifyContent:"flex-end",gap:8}}>
        <Btn onClick={()=>{setMessages(p=>[{id:"msg"+Date.now(),date:TODAY,...form},...p]);setForm({subject:"",body:"",targets:"all",type:"info",pinned:false});setCompose(false);}} disabled={!form.subject.trim()||!form.body.trim()}>📣 Publier</Btn>
      </div>
    </div>}
    {messages.length===0
      ?<div style={{...card,textAlign:"center",padding:36}}><div style={{fontSize:36}}>📣</div><div style={{fontWeight:600,color:"#374151",marginTop:8}}>Aucune annonce publiée</div></div>
      :[...messages].sort((a,b)=>(b.pinned?1:0)-(a.pinned?1:0)||b.date.localeCompare(a.date)).map(msg=>{
        const s=MST[msg.type]||MST.info;
        const dest=msg.targets==="all"?`Tous (${prestataires.length})`:Array.isArray(msg.targets)?prestataires.filter(p=>msg.targets.includes(p.id)).map(p=>p.name.split(" ")[0]).join(", "):"Tous";
        return <div key={msg.id} style={{...card,borderLeft:`4px solid ${s.bd}`,position:"relative"}}>
          {msg.pinned&&<span style={{position:"absolute",top:12,right:40,fontSize:11,color:B.primary,fontWeight:600}}>📌</span>}
          <button onClick={()=>setMessages(p=>p.filter(m=>m.id!==msg.id))} style={{position:"absolute",top:10,right:10,background:"none",border:"none",color:"#9CA3AF",cursor:"pointer",fontSize:14}}>✕</button>
          <div style={{display:"flex",gap:7,alignItems:"center",marginBottom:5}}><span style={{fontSize:15}}>{s.icon}</span><span style={{fontWeight:700,fontSize:13,color:"#111827"}}>{msg.subject}</span></div>
          <div style={{fontSize:11,color:"#9CA3AF",marginBottom:8}}>{msg.date} · <span style={{color:B.primary,fontWeight:500}}>→ {dest}</span></div>
          <div style={{fontSize:13,color:"#374151",lineHeight:1.7,whiteSpace:"pre-wrap",background:"#F9FAFB",borderRadius:8,padding:"9px 12px"}}>{msg.body}</div>
        </div>;
      })}
  </div>;
}

// ─── MANAGER APP ──────────────────────────────────────────────────────────────
function ManagerApp({onLogout,ppls,setPpls,documents,setDocuments,properties,setProperties,prestataires,setPrestataires,messages,setMessages,produits,setProduits,stocks,setStocks}) {
  const [tab,setTab]=useState("dashboard");
  const [tasks,setTasks]=useState([
    {id:1,prop:"villa-azur",   title:"Robinet salle de bain qui fuit", q:null},
    {id:2,prop:"apt-lumiere",  title:"Chaudière en panne",             q:null},
    {id:3,prop:"mas-provence", title:"Piscine à désinfecter",          q:null},
    {id:4,prop:"loft-bordeaux",title:"Wifi ne fonctionne plus",        q:null},
    {id:5,prop:"chalet-mont",  title:"Radiateur chambre froide",       q:null},
  ]);
  const [aiLog,setAiLog]=useState([]);
  const [classified,setClassified]=useState(false);
  const [aiLoading,setAiLoading]=useState(false);
  const [selPrest,setSelPrest]=useState(null);
  const [selProp,setSelProp]=useState(null);
  const [addPrestModal,setAddPrestModal]=useState(false);
  const [addPropModal,setAddPropModal]=useState(false);
  const [newPplModal,setNewPplModal]=useState(false);
  const [emailModal,setEmailModal]=useState(null);
  const [emailContent,setEmailContent]=useState("");
  const [emailLoading,setEmailLoading]=useState(false);
  const [candidateModal,setCandidateModal]=useState(null);
  const [vigLoading,setVigLoading]=useState(false);
  const [vigPoints,setVigPoints]=useState([]);
  const [newPplForm,setNewPplForm]=useState({property:"",date:"",timeSlot:"",prestataire:"",guest:"",guestCount:1,nextCheckin:""});
  const [prestForm,setPrestForm]=useState({name:"",role:"",email:"",phone:"",pin:"",properties:[]});
  const [propForm,setPropForm]=useState({name:"",address:"",access:"",duration:120,lodgifyKey:""});
  const [pplFilter,setPplFilter]=useState("all");
  const [docDest,setDocDest]=useState("all");
  const docRef=useRef(null);

  const gp=id=>properties.find(p=>p.id===id);
  const gr=id=>prestataires.find(p=>p.id===id);
  const lodgifyPending=ppls.filter(m=>m.source==="lodgify"&&m.status==="disponible").length;
  const candidaturesPending=ppls.filter(m=>m.status==="disponible"&&(m.candidates||[]).length>0).length;
  const stockAlerts=properties.reduce((n,prop)=>n+produits.filter(p=>{
    const s=stocks[prop.id]?.[p.id];
    if(!s)return false;
    return p.type==="level"?(s.level==="1/4"||s.level==="empty"):s.qty<p.min;
  }).length,0);

  const TABS=[
    {id:"dashboard",   icon:"▦",  label:"Dashboard"},
    {id:"matrix",      icon:"⊞",  label:"Matrice"},
    {id:"ppls",        icon:"📋", label:"PPL",         badge:lodgifyPending+candidaturesPending},
    {id:"properties",  icon:"⌂",  label:"Logements"},
    {id:"prestataires",icon:"👤", label:"Prestataires"},
    {id:"stocks",      icon:"📦", label:"Stocks",      badge:stockAlerts},
    {id:"documents",   icon:"📁", label:"Documents"},
    {id:"annonces",    icon:"📣", label:"Annonces"},
  ];

  const QUADS={Q1:{bg:"#FFF1F1",bd:"#FCA5A5",badge:"#EF4444",text:"#991B1B",label:"Urgent & Important"},Q2:{bg:"#FFFBEB",bd:"#FCD34D",badge:"#F59E0B",text:"#92400E",label:"Important, pas urgent"},Q3:{bg:"#EFF6FF",bd:"#93C5FD",badge:"#3B82F6",text:"#1E40AF",label:"Urgent, peu important"},Q4:{bg:"#F9FAFB",bd:"#D1D5DB",badge:"#9CA3AF",text:"#4B5563",label:"Ni urgent ni important"}};

  const classify=async()=>{
    setAiLoading(true);setAiLog([]);setAiLog(p=>[...p,"🤖 Analyse en cours..."]);
    try{
      const txt=await callClaude(`Classifie selon Eisenhower. JSON sans backticks. Format:[{"id":N,"q":"Q1"|"Q2"|"Q3"|"Q4","reason":"courte"}]`,`Tâches:${JSON.stringify(tasks.map(t=>({id:t.id,logement:gp(t.prop)?.name,titre:t.title})))}`);
      const list=JSON.parse(txt);
      setTasks(prev=>prev.map(t=>{const c=list.find(x=>x.id===t.id);return c?{...t,...c}:t;}));
      setClassified(true);setAiLog(p=>[...p,"✅ Classifié"]);
    }catch(e){setAiLog(p=>[...p,"❌ "+e.message]);}
    setAiLoading(false);
  };

  const genVig=async(propId)=>{
    setVigLoading(true);setVigPoints([]);
    try{const t=await callClaude(`Points de vigilance. JSON. Format:{"points":["string"]} max 5.`,`Logement:"${gp(propId)?.name}". Historique:${JSON.stringify(PPL_HIST[propId]||[])}.`);setVigPoints(JSON.parse(t).points||[]);}
    catch{setVigPoints([]);}
    setVigLoading(false);
  };

  const filtered=pplFilter==="all"?ppls:ppls.filter(m=>m.status===pplFilter);

  const side={width:216,background:"#fff",borderRight:"1px solid #E5EDF2",position:"fixed",top:0,left:0,bottom:0,display:"flex",flexDirection:"column",zIndex:100,fontFamily:"'DM Sans','Segoe UI',sans-serif"};
  const navBtn=(active)=>({display:"flex",alignItems:"center",gap:8,padding:"9px 14px",margin:"1px 8px",borderRadius:9,border:"none",width:"calc(100% - 16px)",textAlign:"left",cursor:"pointer",fontSize:13,fontWeight:active?600:400,color:active?B.dark:"#5B6B7C",background:active?B.light:"transparent"});

  return <div style={{minHeight:"100vh",background:"#F4F7F9",display:"flex",fontFamily:"'DM Sans','Segoe UI',sans-serif"}}>
    <div style={side}>
      <div style={{padding:"20px 16px 12px",borderBottom:"1px solid #E5EDF2"}}><Logo/><div style={{fontSize:9,color:"#9CAAB8",textTransform:"uppercase",letterSpacing:"0.14em",marginTop:3,fontWeight:500}}>Gestionnaire</div></div>
      <div style={{padding:"8px 0",flex:1,overflowY:"auto"}}>
        {TABS.map(t=>(
          <button key={t.id} style={navBtn(tab===t.id)} onClick={()=>setTab(t.id)}>
            <span style={{color:tab===t.id?B.primary:"#9CAAB8",fontSize:13}}>{t.icon}</span>{t.label}
            {(t.badge||0)>0&&<span style={{marginLeft:"auto",background:"#EF4444",color:"#fff",fontSize:10,fontWeight:700,borderRadius:10,padding:"1px 6px"}}>{t.badge}</span>}
          </button>
        ))}
      </div>
      <div style={{padding:"8px 12px",borderTop:"1px solid #E5EDF2",display:"flex",gap:7,flexWrap:"wrap",alignItems:"center"}}>
        {[["#22C55E","Lodgify"],["#22C55E","Gmail"],["#22C55E","Make"]].map(([c,l])=>(<div key={l} style={{display:"flex",alignItems:"center",gap:3,fontSize:10,color:"#9CAAB8"}}><div style={{width:5,height:5,borderRadius:"50%",background:c}}/>{l}</div>))}
        <button onClick={onLogout} style={{marginLeft:"auto",background:"none",border:"none",color:"#9CA3AF",fontSize:11,cursor:"pointer"}}>Sortir</button>
      </div>
    </div>

    <div style={{marginLeft:216,padding:"26px 30px",flex:1}}>

      {/* ── DASHBOARD ── */}
      {tab==="dashboard"&&<div>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:20}}>
          <div><div style={{fontSize:20,fontWeight:700,color:"#111827"}}>Tableau de bord</div><div style={{fontSize:13,color:"#6B7280"}}>{new Date().toLocaleDateString("fr-FR",{weekday:"long",year:"numeric",month:"long",day:"numeric"})}</div></div>
          <Btn onClick={classify} disabled={aiLoading}>{aiLoading?"⏳ Analyse...":"✦ Classifier avec l'IA"}</Btn>
        </div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)",gap:12,marginBottom:18}}>
          {[{label:"Tâches maintenance",value:tasks.length,main:true},{label:"PPL aujourd'hui",value:ppls.filter(m=>m.date===TODAY).length,c:"#10B981"},{label:"PPL en attente",value:ppls.filter(m=>m.status==="proposée").length,c:"#F59E0B"},{label:"Alertes stock",value:stockAlerts,c:"#EF4444"}].map((s,i)=>(
            <div key={i} style={{background:s.main?B.primary:"#fff",border:s.main?"none":"1px solid #E5EDF2",borderRadius:12,padding:"14px 16px"}}>
              {!s.main&&<div style={{width:20,height:3,borderRadius:2,background:s.c,marginBottom:7}}/>}
              <div style={{fontSize:26,fontWeight:800,color:s.main?"#fff":"#111827",lineHeight:1,marginBottom:1}}>{s.value}</div>
              <div style={{fontSize:10,color:s.main?"#ffffffaa":"#9CA3AF",textTransform:"uppercase",letterSpacing:"0.08em"}}>{s.label}</div>
            </div>
          ))}
        </div>
        {aiLog.length>0&&<div style={{background:"#F0F9FF",border:"1px solid #BAE6FD",borderRadius:8,padding:"9px 14px",fontFamily:"monospace",fontSize:12,color:B.dark,marginBottom:14}}>{aiLog.map((l,i)=><div key={i}>{l}</div>)}</div>}
        <div style={card}>
          <div style={ct}>Tâches de maintenance</div>
          {tasks.map(t=>{const Q=QUADS[t.q];return(
            <div key={t.id} style={{display:"flex",alignItems:"center",gap:9,padding:"8px 11px",background:Q?Q.bg:"#F9FAFB",borderRadius:8,marginBottom:5,borderLeft:`3px solid ${Q?Q.bd:"#E5E7EB"}`}}>
              <span style={{fontSize:13,flex:1,color:"#1F2937"}}>{t.title}</span>
              <span style={{fontSize:11,color:"#9CA3AF"}}>{gp(t.prop)?.name}</span>
              {Q?<Bdg color={Q.badge} bg={Q.bg}>{Q.label}</Bdg>:<Bdg color="#9CA3AF" bg="#F3F4F6">Non classé</Bdg>}
            </div>
          );})}
        </div>
      </div>}

      {/* ── MATRICE ── */}
      {tab==="matrix"&&<div>
        <div style={{fontSize:20,fontWeight:700,color:"#111827",marginBottom:4}}>Matrice d'Eisenhower</div>
        <div style={{fontSize:13,color:"#6B7280",marginBottom:16}}>{classified?"✓ Mis à jour par l'IA":"⚠ Classifiez depuis le Dashboard"}</div>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:12}}>
          {Object.entries(QUADS).map(([key,val])=>{
            const byQ=tasks.filter(t=>t.q===key);
            return <div key={key} style={{background:"#fff",border:`1px solid ${val.bd}`,borderTop:`4px solid ${val.badge}`,borderRadius:12,padding:16,minHeight:130}}>
              <div style={{fontSize:11,fontWeight:700,color:val.text,textTransform:"uppercase",marginBottom:10,display:"flex",justifyContent:"space-between"}}><span>{val.label}</span><Bdg color={val.badge} bg={val.bg}>{byQ.length}</Bdg></div>
              {byQ.length===0?<div style={{fontSize:12,color:"#D1D5DB",fontStyle:"italic"}}>Aucune tâche</div>
              :byQ.map(t=><div key={t.id} style={{background:val.bg,borderRadius:7,padding:"6px 9px",marginBottom:4}}><div style={{fontSize:12,color:"#1F2937",fontWeight:500}}>{t.title}</div><div style={{fontSize:10,color:"#9CA3AF"}}>{gp(t.prop)?.name}{t.reason?` · ${t.reason}`:""}</div></div>)}
            </div>;
          })}
        </div>
      </div>}

      {/* ── PPL ── */}
      {tab==="ppls"&&<div>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:20}}>
          <div><div style={{fontSize:20,fontWeight:700,color:"#111827"}}>PPL</div><div style={{fontSize:13,color:"#6B7280"}}>{lodgifyPending>0?`🔵 ${lodgifyPending} Lodgify en attente`:"Planning des prestations"}</div></div>
          <div style={{display:"flex",gap:8}}>
            {lodgifyPending>0&&<div style={{background:"#EFF6FF",border:"1px solid #93C5FD",borderRadius:9,padding:"9px 12px",fontSize:12,color:"#1E40AF",fontWeight:500}}>🔄 {lodgifyPending}</div>}
            <Btn onClick={()=>{setNewPplModal(true);setVigPoints([]);}}>+ Nouveau PPL</Btn>
          </div>
        </div>
        <div style={{display:"flex",gap:7,marginBottom:14}}>
          {[["all","Tous"],["disponible","Disponibles"],["acceptée","Pris en charge"],["terminée","Terminés"]].map(([vv,l])=>(
            <button key={vv} onClick={()=>setPplFilter(vv)} style={{padding:"7px 13px",borderRadius:8,border:pplFilter===vv?"none":"1px solid #E5EDF2",background:pplFilter===vv?B.primary:"#fff",color:pplFilter===vv?"#fff":"#6B7280",fontSize:12,fontWeight:pplFilter===vv?600:400,cursor:"pointer"}}>
              {l}{vv!=="all"&&` (${ppls.filter(m=>m.status===vv).length})`}
            </button>
          ))}
        </div>
        <div style={{display:"grid",gap:8}}>
          {filtered.map(m=>{const prop=gp(m.property),prest=gr(m.prestataire),cands=m.candidates||[];return(
            <div key={m.id} style={{...card,marginBottom:0,display:"flex",alignItems:"center",gap:11}}>
              <div style={{width:4,alignSelf:"stretch",borderRadius:2,background:m.source==="lodgify"?B.primary:"#F59E0B",flexShrink:0}}/>
              <div style={{flex:1}}>
                <div style={{display:"flex",alignItems:"center",gap:7,marginBottom:3}}>
                  <span style={{fontWeight:700,fontSize:13,color:"#111827"}}>{prop?.name}</span>
                  <Bdg color={m.status==="disponible"?"#8B5CF6":m.status==="acceptée"?"#10B981":"#6B7280"} bg={m.status==="disponible"?"#F5F3FF":m.status==="acceptée"?"#ECFDF5":"#F3F4F6"}>{m.status==="disponible"?"Disponible":m.status==="acceptée"?"Attribué":m.status}</Bdg>
                  {cands.length>0&&<span onClick={()=>setCandidateModal(m)} style={{background:"#FFF7ED",border:"1px solid #FB923C",borderRadius:18,padding:"2px 9px",fontSize:10,fontWeight:700,color:"#C2410C",cursor:"pointer"}}>👋 {cands.length} candidat{cands.length>1?"s":""} — Valider →</span>}
                </div>
                <div style={{display:"flex",gap:12,fontSize:11,color:"#6B7280"}}>
                  <span>📅 {m.date} {m.timeSlot}</span><span>👥 {m.guestCount||1} voy.</span><span>🏃 {m.guest}</span>
                  {prest&&<span style={{color:B.primary,fontWeight:500}}>👤 {prest.name.split(" ")[0]}</span>}
                  {!m.prestataire&&cands.length===0&&<span style={{color:"#8B5CF6",fontWeight:500}}>⬡ En attente de candidatures</span>}
                </div>
              </div>
              {prest&&<Av name={prest.name} size={26}/>}
              <Btn v="outline" onClick={async()=>{setEmailModal(m);setEmailLoading(true);try{const t=await callClaude("Assistant LOMMAX. Email rapport PPL concis.",`PPL ${prop?.name}. Départ:${m.guest}. Check-in:${m.nextCheckin}.`);setEmailContent(t);}catch{setEmailContent("Erreur");}setEmailLoading(false);}} sx={{padding:"6px 10px",fontSize:11}}>✉</Btn>
            </div>
          );})}
        </div>

        {/* ── Modal validation candidatures ── */}
        {candidateModal&&<Modal onClose={()=>setCandidateModal(null)} w="min(480px,95vw)">
          <div style={{fontWeight:700,fontSize:16,color:"#111827",marginBottom:4}}>👋 Valider une candidature</div>
          <div style={{fontSize:13,color:"#6B7280",marginBottom:16}}>
            {gp(candidateModal.property)?.name} · {candidateModal.date} à {candidateModal.timeSlot} · {candidateModal.guestCount||1} voy.
          </div>
          <div style={{marginBottom:16}}>
            {(candidateModal.candidates||[]).length===0
              ?<div style={{fontSize:13,color:"#9CA3AF",fontStyle:"italic",textAlign:"center",padding:20}}>Aucune candidature pour l'instant</div>
              :(candidateModal.candidates||[]).map(pid=>{
                const p=gr(pid);
                if(!p)return null;
                const scores=Object.values(PPL_HIST).flat().filter(h=>h.by===pid).map(h=>h.score);
                const avg=scores.length?Math.round(scores.reduce((a,b)=>a+b,0)/scores.length):null;
                const myPpls_=ppls.filter(m=>m.prestataire===pid&&m.status==="acceptée").length;
                return <div key={pid} style={{display:"flex",alignItems:"center",gap:12,padding:"12px 14px",background:"#F9FAFB",borderRadius:10,marginBottom:8,border:"1px solid #E5EDF2"}}>
                  <Av name={p.name} size={40}/>
                  <div style={{flex:1}}>
                    <div style={{fontWeight:700,fontSize:14,color:"#111827"}}>{p.name}</div>
                    <div style={{fontSize:11,color:"#9CA3AF",marginTop:1}}>{p.role}</div>
                    <div style={{display:"flex",gap:6,marginTop:5}}>
                      {avg&&<Bdg color={avg>=80?"#10B981":avg>=60?"#F59E0B":"#EF4444"} bg={avg>=80?"#ECFDF5":avg>=60?"#FFFBEB":"#FFF1F1"}>⭐ {avg}/100</Bdg>}
                      <Bdg color={B.primary} bg={B.light}>{myPpls_} PPL en cours</Bdg>
                    </div>
                  </div>
                  <Btn onClick={()=>{
                    setPpls(prev=>prev.map(m=>m.id===candidateModal.id?{...m,prestataire:pid,status:"acceptée",candidates:[]}:m));
                    setCandidateModal(null);
                  }} sx={{padding:"8px 14px",fontSize:12}}>✓ Valider</Btn>
                </div>;
              })}
          </div>
          <div style={{display:"flex",gap:8}}>
            <Btn v="outline" onClick={()=>setCandidateModal(null)} sx={{flex:1}}>Fermer</Btn>
          </div>
        </Modal>}


        {newPplModal&&<Modal onClose={()=>setNewPplModal(false)}>
          <div style={{fontWeight:700,fontSize:15,marginBottom:16}}>+ Nouveau PPL</div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12}}>
            <Fld label="Logement *"><Sel value={newPplForm.property} onChange={v=>{setNewPplForm(p=>({...p,property:v}));setVigPoints([]);}}><option value="">Choisir...</option>{properties.map(p=><option key={p.id} value={p.id}>{p.name}</option>)}</Sel></Fld>
            <Fld label="Date *"><Inp type="date" value={newPplForm.date} onChange={v=>setNewPplForm(p=>({...p,date:v}))}/></Fld>
            <Fld label="Heure *"><Inp type="time" value={newPplForm.timeSlot} onChange={v=>setNewPplForm(p=>({...p,timeSlot:v}))}/></Fld>
            <Fld label="Locataire"><Inp value={newPplForm.guest} onChange={v=>setNewPplForm(p=>({...p,guest:v}))} placeholder="Nom..."/></Fld>
            <Fld label="Check-in suivant"><Inp type="time" value={newPplForm.nextCheckin} onChange={v=>setNewPplForm(p=>({...p,nextCheckin:v}))}/></Fld>
            <Fld label="Nb voyageurs"><Inp type="number" value={String(newPplForm.guestCount)} onChange={v=>setNewPplForm(p=>({...p,guestCount:parseInt(v)||1}))}/></Fld>
          </div>
          <div style={{marginBottom:12}}>
            <Fld label="Attribuer à (facultatif)">
              <Sel value={newPplForm.prestataire} onChange={v=>setNewPplForm(p=>({...p,prestataire:v}))}>
                <option value="">⬡ Open — visible par tous les prestataires qualifiés</option>
                {prestataires.map(p=><option key={p.id} value={p.id}>{p.name}</option>)}
              </Sel>
            </Fld>
            <div style={{fontSize:11,color:"#9CA3AF",marginTop:3}}>Sans attribution, le PPL est visible dans le pool de toutes les personnes assignées à ce logement.</div>
          </div>
          {newPplForm.property&&<div style={{marginBottom:12}}>
            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:7}}>
              <div style={{fontSize:11,fontWeight:600,color:"#6B7280",textTransform:"uppercase"}}>Points de vigilance IA</div>
              <Btn v="ghost" onClick={()=>genVig(newPplForm.property)} disabled={vigLoading} sx={{fontSize:12,padding:"5px 9px"}}>{vigLoading?"⏳":"✦ Générer"}</Btn>
            </div>
            {vigPoints.length>0?<div style={{background:"#FFFBEB",border:"1px solid #FCD34D",borderRadius:9,padding:11}}>{vigPoints.map((p,i)=><div key={i} style={{display:"flex",gap:6,marginBottom:3}}><span style={{color:"#F59E0B",fontWeight:700}}>⚠</span><span style={{fontSize:13,color:"#374151"}}>{p}</span></div>)}</div>
            :<div style={{background:"#F9FAFB",borderRadius:9,padding:10,fontSize:12,color:"#9CA3AF",textAlign:"center",border:"1px dashed #D1DAE3"}}>Cliquez "✦ Générer"</div>}
          </div>}
          <div style={{display:"flex",gap:8}}><Btn v="outline" onClick={()=>setNewPplModal(false)}>Annuler</Btn><Btn onClick={()=>{setPpls(p=>[{id:"ppl"+Date.now(),...newPplForm,status:newPplForm.prestataire?"proposée":"disponible",source:"manuel",vigilancePoints:vigPoints},...p]);setNewPplModal(false);setNewPplForm({property:"",date:"",timeSlot:"",prestataire:"",guest:"",guestCount:1,nextCheckin:""});setVigPoints([]);}} disabled={!newPplForm.property||!newPplForm.date} sx={{flex:1}}>📤 Publier</Btn></div>
        </Modal>}
        {emailModal&&<Modal onClose={()=>{setEmailModal(null);setEmailContent("");}}>
          <div style={{display:"flex",justifyContent:"space-between",marginBottom:12}}><div style={{fontWeight:700,fontSize:14}}>✉ Rapport — {gp(emailModal.property)?.name}</div><Btn v="outline" onClick={()=>{setEmailModal(null);setEmailContent("");}} sx={{padding:"5px 9px",fontSize:11}}>✕</Btn></div>
          {emailLoading?<div style={{color:B.primary,textAlign:"center",padding:20}}>🤖 Rédaction...</div>:emailContent&&<div style={{whiteSpace:"pre-wrap",fontSize:13,lineHeight:1.8,color:"#374151",background:"#F9FAFB",padding:12,borderRadius:9,border:"1px solid #E5E7EB"}}>{emailContent}</div>}
          {emailContent&&<Btn onClick={()=>navigator.clipboard.writeText(emailContent)} sx={{marginTop:10}}>Copier</Btn>}
        </Modal>}
      </div>}

      {/* ── LOGEMENTS ── */}
      {tab==="properties"&&<div>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:20}}>
          <div><div style={{fontSize:20,fontWeight:700,color:"#111827"}}>Logements</div><div style={{fontSize:13,color:"#6B7280"}}>{properties.length} logements gérés</div></div>
          <Btn onClick={()=>setAddPropModal(true)}>+ Ajouter</Btn>
        </div>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:12}}>
          {properties.map(prop=>{
            const lastScore=(PPL_HIST[prop.id]||[])[0]?.score;
            return <div key={prop.id} style={{...card,cursor:"pointer"}} onClick={()=>setSelProp(prop)}>
              <div style={{display:"flex",justifyContent:"space-between",marginBottom:9}}><div><div style={{fontWeight:700,fontSize:14,color:"#111827"}}>{prop.name}</div><div style={{fontSize:11,color:"#9CA3AF"}}>{prop.address}</div></div>{lastScore&&<SD score={lastScore}/>}</div>
              <div style={{background:B.light,borderRadius:8,padding:"7px 11px",marginBottom:7}}><div style={{fontSize:10,color:B.dark,fontWeight:600,textTransform:"uppercase",marginBottom:1}}>🔑 Accès</div><div style={{fontSize:12,color:B.dark}}>{prop.access}</div></div>
              <div style={{display:"flex",justifyContent:"space-between",fontSize:11,color:"#9CA3AF"}}><span>⏱ {prop.duration} min</span><span style={{color:B.mid,fontWeight:500}}>🔌 API</span></div>
            </div>;
          })}
        </div>
        {selProp&&<Modal onClose={()=>setSelProp(null)}>
          <div style={{display:"flex",justifyContent:"space-between",marginBottom:16}}><div><div style={{fontWeight:800,fontSize:18,color:"#111827"}}>{selProp.name}</div><div style={{fontSize:12,color:"#9CA3AF"}}>{selProp.address}</div></div><Btn v="outline" onClick={()=>setSelProp(null)} sx={{padding:"5px 9px",fontSize:11}}>✕</Btn></div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9,marginBottom:14}}>{[["🔑 Accès",selProp.access],["⏱ Durée",`${selProp.duration} min`],["🔌 Lodgify","lod_●●●●●●"],["📍",selProp.address]].map(([l,v])=>(<div key={l} style={{background:"#F9FAFB",borderRadius:9,padding:"9px 12px"}}><div style={{fontSize:10,color:"#9CA3AF",marginBottom:2}}>{l}</div><div style={{fontSize:13,fontWeight:600,color:"#111827"}}>{v}</div></div>))}</div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:14}}>
            <div><div style={{fontSize:11,fontWeight:600,color:"#9CA3AF",textTransform:"uppercase",marginBottom:7}}>Historique PPL</div>{(PPL_HIST[selProp.id]||[]).map((h,i)=><div key={i} style={{display:"flex",gap:9,alignItems:"center",padding:"6px 0",borderBottom:"1px solid #F9FAFB"}}><SD score={h.score}/><div><div style={{fontSize:12,fontWeight:600,color:"#374151"}}>{h.date}</div><div style={{fontSize:11,color:"#9CA3AF"}}>{gr(h.by)?.name}</div></div></div>)}</div>
            <div><div style={{fontSize:11,fontWeight:600,color:"#9CA3AF",textTransform:"uppercase",marginBottom:7}}>Réservations</div>{(UPCOMING[selProp.id]||[]).map((r,i)=><div key={i} style={{display:"flex",justifyContent:"space-between",fontSize:13,color:"#374151",padding:"5px 0",borderBottom:"1px solid #F9FAFB"}}><span>{r.date}</span><span style={{fontWeight:600}}>{r.guests} voy.</span></div>)}</div>
          </div>
        </Modal>}
        {addPropModal&&<Modal onClose={()=>setAddPropModal(false)}>
          <div style={{fontWeight:700,fontSize:15,marginBottom:16}}>+ Ajouter un logement</div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12}}>
            <Fld label="Nom *"><Inp value={propForm.name} onChange={v=>setPropForm(p=>({...p,name:v}))} placeholder="Villa Azur"/></Fld>
            <Fld label="Adresse *"><Inp value={propForm.address} onChange={v=>setPropForm(p=>({...p,address:v}))}/></Fld>
            <Fld label="Accès *"><Inp value={propForm.access} onChange={v=>setPropForm(p=>({...p,access:v}))}/></Fld>
            <Fld label="Durée PPL (min)"><Inp type="number" value={String(propForm.duration)} onChange={v=>setPropForm(p=>({...p,duration:parseInt(v)||120}))}/></Fld>
            <div style={{gridColumn:"1/-1"}}><Fld label="Clé API Lodgify"><Inp value={propForm.lodgifyKey} onChange={v=>setPropForm(p=>({...p,lodgifyKey:v}))} placeholder="lod_xxxxx"/></Fld></div>
          </div>
          <div style={{display:"flex",gap:8}}><Btn v="outline" onClick={()=>setAddPropModal(false)}>Annuler</Btn><Btn onClick={()=>{setProperties(prev=>[...prev,{...propForm,id:propForm.name.toLowerCase().replace(/[^a-z0-9]/g,"-")}]);setAddPropModal(false);setPropForm({name:"",address:"",access:"",duration:120,lodgifyKey:""}); }} disabled={!propForm.name||!propForm.address} sx={{flex:1}}>Ajouter</Btn></div>
        </Modal>}
      </div>}

      {/* ── PRESTATAIRES ── */}
      {tab==="prestataires"&&<div>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:20}}>
          <div><div style={{fontSize:20,fontWeight:700,color:"#111827"}}>Prestataires</div><div style={{fontSize:13,color:"#6B7280"}}>Cliquez pour voir la fiche complète</div></div>
          <Btn onClick={()=>setAddPrestModal(true)}>+ Ajouter</Btn>
        </div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:12}}>
          {prestataires.map(p=>{
            const pm=ppls.filter(m=>m.prestataire===p.id&&m.status!=="terminée");
            const scores=Object.values(PPL_HIST).flat().filter(h=>h.by===p.id).map(h=>h.score);
            const avg=scores.length>0?Math.round(scores.reduce((a,b)=>a+b,0)/scores.length):null;
            return <div key={p.id} style={{...card,cursor:"pointer"}} onClick={()=>setSelPrest(p)}>
              <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:10}}><Av name={p.name} size={40}/><div><div style={{fontWeight:700,fontSize:13,color:"#111827"}}>{p.name}</div><div style={{fontSize:11,color:"#9CA3AF"}}>{p.role}</div></div></div>
              <div style={{display:"flex",gap:6,marginBottom:8,flexWrap:"wrap"}}><Bdg color="#10B981" bg="#ECFDF5">Actif</Bdg><Bdg color={B.primary} bg={B.light}>{pm.length} PPL</Bdg>{avg&&<Bdg color={avg>=80?"#10B981":avg>=60?"#F59E0B":"#EF4444"} bg={avg>=80?"#ECFDF5":avg>=60?"#FFFBEB":"#FFF1F1"}>{avg}/100</Bdg>}</div>
              {p.properties?.map(pid=><div key={pid} style={{display:"flex",alignItems:"center",gap:5,marginBottom:2}}><div style={{width:4,height:4,borderRadius:"50%",background:B.primary}}/><span style={{fontSize:11,color:"#374151"}}>{gp(pid)?.name}</span></div>)}
              <div style={{marginTop:8,paddingTop:8,borderTop:"1px solid #F3F4F6",fontSize:11,color:B.mid,fontWeight:500}}>Voir la fiche →</div>
            </div>;
          })}
        </div>
        {selPrest&&<Modal onClose={()=>setSelPrest(null)} w="min(700px,96vw)">
          <div style={{display:"flex",alignItems:"center",gap:14,marginBottom:20,paddingBottom:18,borderBottom:"1px solid #F3F4F6"}}>
            <Av name={selPrest.name} size={52}/>
            <div style={{flex:1}}>
              <div style={{fontWeight:800,fontSize:18,color:"#111827"}}>{selPrest.name}</div>
              <div style={{fontSize:12,color:"#9CA3AF"}}>{selPrest.role}</div>
              <div style={{display:"flex",gap:7,marginTop:7}}><Bdg color="#10B981" bg="#ECFDF5">Actif</Bdg><Bdg color={B.primary} bg={B.light}>{ppls.filter(m=>m.prestataire===selPrest.id&&m.status==="terminée").length} PPL terminés</Bdg></div>
            </div>
            <Btn v="outline" onClick={()=>setSelPrest(null)} sx={{padding:"5px 9px",fontSize:11}}>✕</Btn>
          </div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9,marginBottom:16}}>{[["📧",selPrest.email||"—"],["📞",selPrest.phone||"—"],["🔑 PIN","●●●● ("+selPrest.pin+")"],["🏠 Logements",selPrest.properties?.map(id=>gp(id)?.name).join(", ")||"—"]].map(([l,v])=>(<div key={l} style={{background:"#F9FAFB",borderRadius:9,padding:"9px 12px"}}><div style={{fontSize:10,color:"#9CA3AF",marginBottom:2}}>{l}</div><div style={{fontSize:13,fontWeight:600,color:"#111827"}}>{v}</div></div>))}</div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:14}}>
            <div><div style={{fontSize:11,fontWeight:600,color:"#9CA3AF",textTransform:"uppercase",marginBottom:7}}>Historique PPL</div>{Object.entries(PPL_HIST).flatMap(([pid,arr])=>arr.filter(h=>h.by===selPrest.id).map(h=>({...h,pid}))).sort((a,b)=>b.date.localeCompare(a.date)).slice(0,5).map((h,i)=><div key={i} style={{display:"flex",gap:9,alignItems:"center",padding:"7px 0",borderBottom:"1px solid #F9FAFB"}}><SD score={h.score}/><div><div style={{fontSize:12,fontWeight:600,color:"#374151"}}>{gp(h.pid)?.name}</div><div style={{fontSize:11,color:"#9CA3AF"}}>{h.date}</div></div>{h.issues.length===0?<span style={{fontSize:11,color:"#10B981"}}>✓</span>:<span style={{fontSize:11,color:"#F59E0B"}}>{h.issues.length} pt</span>}</div>)}</div>
            <div><div style={{fontSize:11,fontWeight:600,color:"#9CA3AF",textTransform:"uppercase",marginBottom:7}}>Documents partagés</div>{documents.filter(d=>d.prestataire===selPrest.id||d.prestataire==="all").map(doc=><div key={doc.id} style={{display:"flex",gap:8,padding:"6px 0",borderBottom:"1px solid #F9FAFB"}}><span style={{fontSize:14}}>{doc.type?.includes("pdf")?"📄":"📝"}</span><span style={{fontSize:12,color:"#374151"}}>{doc.name}</span></div>)}</div>
          </div>
        </Modal>}
        {addPrestModal&&<Modal onClose={()=>setAddPrestModal(false)}>
          <div style={{fontWeight:700,fontSize:15,marginBottom:16}}>+ Ajouter un prestataire</div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12}}>
            <Fld label="Nom *"><Inp value={prestForm.name} onChange={v=>setPrestForm(p=>({...p,name:v}))}/></Fld>
            <Fld label="Rôle"><Inp value={prestForm.role} onChange={v=>setPrestForm(p=>({...p,role:v}))} placeholder="Ménage & Entretien"/></Fld>
            <Fld label="Email"><Inp type="email" value={prestForm.email} onChange={v=>setPrestForm(p=>({...p,email:v}))}/></Fld>
            <Fld label="Téléphone"><Inp value={prestForm.phone} onChange={v=>setPrestForm(p=>({...p,phone:v}))}/></Fld>
            <div style={{gridColumn:"1/-1"}}><Fld label="PIN (4 chiffres) *"><Inp value={prestForm.pin} onChange={v=>setPrestForm(p=>({...p,pin:v.slice(0,4)}))} placeholder="●●●●"/></Fld></div>
          </div>
          <div style={{marginBottom:14}}>
            <div style={{fontSize:11,fontWeight:600,color:"#6B7280",textTransform:"uppercase",marginBottom:7}}>Logements assignés</div>
            <div style={{display:"flex",gap:7,flexWrap:"wrap"}}>
              {properties.map(prop=>{const sel=prestForm.properties?.includes(prop.id);return <button key={prop.id} onClick={()=>setPrestForm(p=>({...p,properties:sel?p.properties.filter(x=>x!==prop.id):[...(p.properties||[]),prop.id]}))} style={{padding:"6px 11px",borderRadius:8,border:`1px solid ${sel?B.primary:"#E5EDF2"}`,background:sel?B.light:"#fff",color:sel?B.dark:"#374151",fontSize:12,cursor:"pointer",fontWeight:sel?600:400}}>{sel?"✓ ":""}{prop.name}</button>;})}
            </div>
          </div>
          <div style={{display:"flex",gap:8}}><Btn v="outline" onClick={()=>setAddPrestModal(false)}>Annuler</Btn><Btn onClick={()=>{setPrestataires(prev=>[...prev,{...prestForm,id:"p"+Date.now()}]);setPrestForm({name:"",role:"",email:"",phone:"",pin:"",properties:[]});setAddPrestModal(false);}} disabled={!prestForm.name||prestForm.pin?.length!==4} sx={{flex:1}}>Ajouter</Btn></div>
        </Modal>}
      </div>}

      {/* ── STOCKS (composant externe) ── */}
      {tab==="stocks"&&<StocksTab produits={produits} setProduits={setProduits} stocks={stocks} setStocks={setStocks} properties={properties} stockAlerts={stockAlerts}/>}

      {/* ── DOCUMENTS ── */}
      {tab==="documents"&&<div>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:20}}>
          <div style={{fontSize:20,fontWeight:700,color:"#111827"}}>Documents</div>
          <Btn onClick={()=>docRef.current?.click()}>+ Importer</Btn>
        </div>
        <input ref={docRef} type="file" multiple accept=".pdf,.doc,.docx" style={{display:"none"}} onChange={e=>Array.from(e.target.files).forEach(file=>setDocuments(p=>[{id:"d"+Date.now()+Math.random(),name:file.name,prestataire:docDest,size:file.size,type:file.type,url:URL.createObjectURL(file),uploadedAt:new Date().toLocaleDateString("fr-FR")},...p]))}/>
        <div style={{...card,marginBottom:14}}>
          <div style={ct}>Destinataire</div>
          <div style={{display:"flex",gap:7,flexWrap:"wrap"}}>{[{id:"all",label:"Tous"},...prestataires.map(p=>({id:p.id,label:p.name}))].map(opt=>(
            <button key={opt.id} onClick={()=>setDocDest(opt.id)} style={{padding:"6px 12px",borderRadius:8,border:docDest===opt.id?"none":"1px solid #E5EDF2",background:docDest===opt.id?B.primary:"#F9FAFB",color:docDest===opt.id?"#fff":"#374151",fontSize:12,fontWeight:docDest===opt.id?600:400,cursor:"pointer"}}>{opt.label}</button>
          ))}</div>
        </div>
        {documents.length===0?<div style={{...card,textAlign:"center",padding:36}}><div style={{fontSize:36}}>📁</div><div style={{fontWeight:600,color:"#374151",marginTop:8}}>Aucun document</div></div>
        :<div style={card}>
          <div style={ct}>Documents ({documents.length})</div>
          {documents.map(doc=>{const prest=doc.prestataire==="all"?null:prestataires.find(p=>p.id===doc.prestataire);return(
            <div key={doc.id} style={{display:"flex",alignItems:"center",gap:10,padding:"9px 0",borderBottom:"1px solid #F3F4F6"}}>
              <div style={{width:30,height:30,borderRadius:8,background:doc.type?.includes("pdf")?"#FFF1F1":"#EFF6FF",display:"flex",alignItems:"center",justifyContent:"center",fontSize:14}}>{doc.type?.includes("pdf")?"📄":"📝"}</div>
              <div style={{flex:1}}><div style={{fontSize:13,fontWeight:600,color:"#111827"}}>{doc.name}</div><div style={{fontSize:11,color:"#9CA3AF"}}>{(doc.size/1024).toFixed(0)} Ko · {doc.uploadedAt}</div></div>
              <Bdg color={prest?B.primary:"#10B981"} bg={prest?B.light:"#ECFDF5"}>{prest?prest.name:"Tous"}</Bdg>
              <Btn v="danger" onClick={()=>setDocuments(p=>p.filter(d=>d.id!==doc.id))} sx={{padding:"4px 8px",fontSize:11}}>✕</Btn>
            </div>
          );})}
        </div>}
      </div>}

      {/* ── ANNONCES (composant externe) ── */}
      {tab==="annonces"&&<AnnoncesTab messages={messages} setMessages={setMessages} prestataires={prestataires}/>}

    </div>
  </div>;
}

// ─── PPL FORM ─────────────────────────────────────────────────────────────────
function PPLForm({ppl,prestataire,properties,onComplete,produits,stocks,onUpdateStocks}) {
  const gp=id=>properties.find(p=>p.id===id);
  const prop=gp(ppl.property);
  const hist=PPL_HIST[ppl.property]||[];
  const upcoming=UPCOMING[ppl.property]||[];
  const [step,setStep]=useState("proprete");
  const [vigPoints,setVigPoints]=useState(ppl.vigilancePoints||[]);
  const [vigLoading,setVigLoading]=useState(false);
  const [vigPhotos,setVigPhotos]=useState({});
  const [genPhotos,setGenPhotos]=useState([]);
  const [analyzing,setAnalyzing]=useState(false);
  const [score,setScore]=useState(null);
  const [feedback,setFeedback]=useState(null);
  const [guide,setGuide]=useState(null);
  const [guideLoading,setGuideLoading]=useState(true);
  const [maintItems,setMaintItems]=useState([{desc:"",photos:[]}]);
  const [qty,setQty]=useState({});
  const [stockPhoto,setStockPhoto]=useState(null);
  const [stockAnalyzing,setStockAnalyzing]=useState(false);
  const [stockAiDone,setStockAiDone]=useState(false);

  useEffect(()=>{
    const init={};
    (produits||[]).forEach(p=>{init[p.id]=(stocks?.[ppl.property]?.[p.id]?.qty)??0;});
    setQty(init);
    loadGuide();
    if(!ppl.vigilancePoints?.length)loadVig();
  },[]);

  async function loadGuide(){
    setGuideLoading(true);
    try{const t=await callClaude(`Guide présentation logement. JSON. Format:{"tips":[{"zone":"s","conseil":"s"}]} max 4.`,`Logement:"${prop?.name}". Historique:${JSON.stringify(hist)}.`);setGuide(JSON.parse(t).tips||[]);}
    catch{setGuide([{zone:"Général",conseil:"Vérifiez chaque pièce du plafond vers le sol."}]);}
    setGuideLoading(false);
  }

  async function loadVig(){
    setVigLoading(true);
    try{const t=await callClaude(`Points de vigilance ménage. JSON. Format:{"points":["s"]} max 5.`,`Logement:"${prop?.name}". Historique:${JSON.stringify(hist)}.`);setVigPoints(JSON.parse(t).points||[]);}
    catch{setVigPoints(["Miroir et vitrages","Literie et oreillers","Kit de bienvenue","Sol et angles","Salle de bain"]);}
    setVigLoading(false);
  }

  async function handleVigPhoto(i,file){
    const preview=URL.createObjectURL(file);
    setVigPhotos(p=>({...p,[i]:{file,preview,analyzing:true,result:null}}));
    try{const raw=await analyzeImg(file,`Inspection logement. JSON. Format:{"ok":bool,"score":0-100,"observation":"1 phrase","correction":"action ou null"}`,`Point:"${vigPoints[i]}". Logement:"${prop?.name}".`);setVigPhotos(p=>({...p,[i]:{file,preview,analyzing:false,result:JSON.parse(raw)}}));}
    catch{setVigPhotos(p=>({...p,[i]:{file,preview,analyzing:false,result:{ok:false,score:0,observation:"Erreur",correction:null}}}));}
  }

  async function analyzeGeneral(){
    if(!genPhotos.length)return;
    setAnalyzing(true);
    const results=[];
    for(const ph of genPhotos){
      try{const raw=await analyzeImg(ph.file,`Expert. JSON. Format:{"score":0-100,"points_corriger":["max 3"]}`,`Photo "${prop?.name}".`);results.push(JSON.parse(raw));}
      catch{results.push({score:0,points_corriger:["Erreur"]});}
    }
    const avg=Math.round(results.reduce((s,r)=>s+(r.score||0),0)/results.length);
    setScore(avg);setFeedback(results);setAnalyzing(false);
  }

  async function analyzeStock(){
    if(!stockPhoto)return;
    setStockAnalyzing(true);
    const format=(produits||[]).reduce((acc,p)=>{acc[p.id]=0;return acc;},{});
    try{
      const raw=await analyzeImg(stockPhoto.file,`Compte consommables. JSON. Format:{"counted":${JSON.stringify(format)}}`,`Produits:${(produits||[]).map(p=>p.label).join(",")}. "${prop?.name}".`);
      const data=JSON.parse(raw);
      setQty(prev=>{const n={...prev};(produits||[]).forEach(p=>{if(data.counted?.[p.id]!=null)n[p.id]=data.counted[p.id];});return n;});
      setStockAiDone(true);
    }catch(e){console.error(e);}
    setStockAnalyzing(false);
  }

  const sc=s=>s>=80?"#10B981":s>=60?"#F59E0B":"#EF4444";
  const cats=[...new Set((produits||[]).map(p=>p.cat))];
  const totalGuests=upcoming.reduce((s,r)=>s+r.guests,0);
  const totalStays=upcoming.length;
  const getReq=p=>(p.perV||0)*totalGuests+(p.perS||0)*totalStays;
  const LEVELS_PPL=["full","3/4","1/2","1/4","empty"];
  const LEVEL_LABELS={"full":"Plein","3/4":"3/4","1/2":"1/2","1/4":"¼ ⚠","empty":"Vide ⚠"};
  const LEVEL_FILLS={"full":100,"3/4":75,"1/2":50,"1/4":25,"empty":0};
  const isLevelAlert=l=>l==="1/4"||l==="empty";

  const STEPS=[{id:"proprete",icon:"🧹",label:"Propreté"},{id:"maintenance",icon:"🔧",label:"Maintenance"},{id:"stock",icon:"📦",label:"Stocks"},{id:"recap",icon:"✅",label:"Récap"}];

  const submit=()=>{
    if(onUpdateStocks){
      const update={};
      (produits||[]).forEach(p=>{
        if(p.type==="level") update[p.id]={level:qty[p.id]||"full",at:TODAY.slice(5),by:prestataire.id};
        else update[p.id]={qty:qty[p.id]??0,at:TODAY.slice(5),by:prestataire.id};
      });
      onUpdateStocks(ppl.property,update);
    }
    onComplete({score,qty});
  };

  return <div>
    <div style={{display:"flex",alignItems:"center",gap:5,marginBottom:20,flexWrap:"wrap"}}>
      {STEPS.map((s,i)=>{const done=STEPS.findIndex(x=>x.id===step)>i,active=step===s.id;return <div key={s.id} style={{display:"flex",alignItems:"center",gap:4}}>
        {i>0&&<div style={{width:18,height:2,background:done?B.primary:"#E5EDF2",borderRadius:2}}/>}
        <div style={{display:"flex",alignItems:"center",gap:4}}>
          <div style={{width:24,height:24,borderRadius:"50%",background:done||active?B.primary:"#E5EDF2",color:done||active?"#fff":"#9CA3AF",fontSize:11,fontWeight:700,display:"flex",alignItems:"center",justifyContent:"center"}}>{done?"✓":s.icon}</div>
          <span style={{fontSize:12,fontWeight:active?700:400,color:active?B.dark:"#9CA3AF"}}>{s.label}</span>
        </div>
      </div>;})}
    </div>

    {step==="proprete"&&<div>
      <div style={{fontWeight:700,fontSize:16,marginBottom:2}}>🧹 Propreté & Présentation</div>
      <div style={{fontSize:13,color:"#6B7280",marginBottom:12}}>{prop?.name} · {ppl.date}</div>
      {guide&&<div style={{...card,background:B.light,border:`1px solid ${B.mid}44`}}>
        <div style={ct}>✦ Guide IA</div>
        {guideLoading?<div style={{fontSize:13,color:B.primary,fontStyle:"italic"}}>⏳ Chargement...</div>
        :guide.map((tip,i)=><div key={i} style={{display:"flex",gap:8,marginBottom:6}}><span style={{background:B.light,color:B.primary,fontWeight:700,fontSize:10,padding:"2px 6px",borderRadius:5,flexShrink:0}}>{tip.zone}</span><span style={{fontSize:13,color:"#374151"}}>{tip.conseil}</span></div>)}
      </div>}
      <div style={card}>
        <div style={ct}>📍 Points de vigilance</div>
        {vigLoading?<div style={{fontSize:13,color:B.primary,fontStyle:"italic"}}>⏳ Génération...</div>
        :vigPoints.map((pt,i)=>{const vp=vigPhotos[i];return <div key={i} style={{border:"1px solid #E5EDF2",borderRadius:10,padding:"11px 13px",marginBottom:8,background:vp?.result?vp.result.ok?"#F0FDF4":"#FFFBEB":"#fff"}}>
          <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:vp?7:0}}>
            <div style={{width:20,height:20,borderRadius:"50%",background:vp?.result?vp.result.ok?"#10B981":B.light:"#F3F4F6",display:"flex",alignItems:"center",justifyContent:"center",fontSize:10,fontWeight:700,color:vp?.result?vp.result.ok?"#fff":B.primary:"#9CA3AF",flexShrink:0}}>{vp?.result?vp.result.ok?"✓":"!":i+1}</div>
            <span style={{fontSize:13,fontWeight:600,color:"#374151",flex:1}}>{pt}</span>
            {!vp&&<label htmlFor={`vp-${ppl.id}-${i}`} style={{fontSize:12,padding:"4px 10px",borderRadius:8,background:B.light,border:`1px solid ${B.mid}`,color:B.dark,cursor:"pointer",fontWeight:500}}>📸</label>}
            {vp&&!vp.analyzing&&<span style={{fontSize:11,fontWeight:700,color:sc(vp.result?.score||0)}}>{vp.result?.score||0}/100</span>}
          </div>
          <input type="file" accept="image/*" id={`vp-${ppl.id}-${i}`} style={{display:"none"}} onChange={e=>{if(e.target.files[0])handleVigPhoto(i,e.target.files[0]);}}/>
          {vp&&<div style={{display:"flex",gap:8,alignItems:"flex-start"}}>
            <img src={vp.preview} alt="" style={{width:60,height:45,objectFit:"cover",borderRadius:7,flexShrink:0}}/>
            {vp.analyzing?<div style={{fontSize:12,color:B.primary,fontStyle:"italic"}}>🤖 Analyse...</div>
            :vp.result&&<div style={{flex:1}}><div style={{fontSize:12,color:"#374151"}}>{vp.result.observation}</div>{!vp.result.ok&&vp.result.correction&&<div style={{fontSize:12,color:"#F59E0B",fontWeight:500}}>→ {vp.result.correction}</div>}</div>}
          </div>}
        </div>;})}
      </div>
      <div style={card}>
        <div style={ct}>📸 Photos générales (min. 4)</div>
        <UpZone onFiles={files=>setGenPhotos(p=>[...p,...Array.from(files).map(f=>({file:f,preview:URL.createObjectURL(f)}))])} label="Ajouter des photos"/>
        <PhotoRow photos={genPhotos} onRemove={i=>setGenPhotos(p=>p.filter((_,j)=>j!==i))}/>
        {genPhotos.length>=2&&!score&&<Btn onClick={analyzeGeneral} disabled={analyzing} sx={{width:"100%",marginTop:10}}>{analyzing?"⏳ Analyse...":"✦ Analyser avec l'IA"}</Btn>}
        {score!=null&&<div style={{marginTop:10,background:score>=80?"#ECFDF5":score>=60?"#FFFBEB":"#FFF1F1",border:`1px solid ${sc(score)}44`,borderRadius:10,padding:"11px 14px",display:"flex",gap:12,alignItems:"center"}}>
          <div style={{width:50,height:50,borderRadius:"50%",background:"#fff",border:`4px solid ${sc(score)}`,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",flexShrink:0}}>
            <div style={{fontSize:15,fontWeight:800,color:sc(score),lineHeight:1}}>{score}</div>
            <div style={{fontSize:9,color:sc(score),fontWeight:600}}>/100</div>
          </div>
          <div><div style={{fontWeight:700,fontSize:13}}>{score>=80?"✅ Excellent !":score>=60?"⚠️ Quelques corrections":"🚨 Corrections requises"}</div>{feedback?.flatMap(r=>r.points_corriger||[]).slice(0,3).map((c,i)=><div key={i} style={{fontSize:12,color:"#374151"}}>→ {c}</div>)}</div>
        </div>}
      </div>
      <Btn onClick={()=>setStep("maintenance")} sx={{width:"100%"}}>Continuer → Maintenance</Btn>
    </div>}

    {step==="maintenance"&&<div>
      <div style={{fontWeight:700,fontSize:16,marginBottom:2}}>🔧 Maintenance</div>
      <div style={{fontSize:13,color:"#6B7280",marginBottom:12}}>Signalez les problèmes nécessitant une intervention</div>
      {maintItems.map((item,i)=>(
        <div key={i} style={{...card,borderLeft:`3px solid ${B.primary}`}}>
          <div style={{display:"flex",justifyContent:"space-between",marginBottom:8}}><span style={{fontWeight:600,fontSize:13}}>Problème {i+1}</span>{maintItems.length>1&&<button onClick={()=>setMaintItems(p=>p.filter((_,j)=>j!==i))} style={{background:"none",border:"none",color:"#EF4444",cursor:"pointer",fontSize:12}}>Suppr.</button>}</div>
          <textarea placeholder="Décrivez précisément..." value={item.desc} onChange={e=>setMaintItems(p=>p.map((x,j)=>j===i?{...x,desc:e.target.value}:x))} style={{width:"100%",background:"#F9FAFB",border:"1px solid #D1DAE3",borderRadius:9,padding:"9px 12px",fontSize:13,outline:"none",resize:"vertical",minHeight:70,marginBottom:9,boxSizing:"border-box",fontFamily:"inherit"}}/>
          <UpZone onFiles={files=>setMaintItems(p=>p.map((x,j)=>j===i?{...x,photos:[...x.photos,...Array.from(files).map(f=>({file:f,preview:URL.createObjectURL(f)}))]}:x))} label="Photos (facultatif)"/>
          <PhotoRow photos={item.photos} onRemove={pi=>setMaintItems(p=>p.map((x,j)=>j===i?{...x,photos:x.photos.filter((_,k)=>k!==pi)}:x))}/>
        </div>
      ))}
      <Btn v="outline" onClick={()=>setMaintItems(p=>[...p,{desc:"",photos:[]}])} sx={{width:"100%",marginBottom:10}}>+ Ajouter un problème</Btn>
      <div style={{...card,background:B.light,border:`1px solid ${B.mid}44`,fontSize:13,color:B.dark,marginBottom:10}}>ℹ️ Les problèmes seront transmis au gestionnaire.</div>
      <div style={{display:"flex",gap:8}}><Btn v="outline" onClick={()=>setStep("proprete")}>← Retour</Btn><Btn onClick={()=>setStep("stock")} sx={{flex:1}}>Stocks →</Btn></div>
    </div>}

    {step==="stock"&&<div>
      <div style={{fontWeight:700,fontSize:16,marginBottom:2}}>📦 Stocks</div>
      <div style={{fontSize:13,color:"#6B7280",marginBottom:12}}>Quantités et niveaux — transmis au gestionnaire.</div>
      {upcoming.length>0&&<div style={{...card,background:B.light,border:`1px solid ${B.mid}44`}}>
        <div style={ct}>Besoins estimés (réservations à venir)</div>
        <div style={{display:"flex",gap:7,flexWrap:"wrap"}}>
          {(produits||[]).filter(p=>p.type==="qty"&&getReq(p)>0).map(p=><div key={p.id} style={{background:"#fff",border:"1px solid #BAE6FD",borderRadius:8,padding:"4px 9px",fontSize:12,color:B.dark}}>{p.icon} <strong>{getReq(p)}</strong> {p.label}</div>)}
        </div>
      </div>}
      {cats.map(cat=>(
        <div key={cat} style={card}>
          <div style={ct}>{cat}</div>
          {(produits||[]).filter(p=>p.cat===cat).map(prod=>{
            if(prod.type==="level"){
              const curLevel=qty[prod.id]||"full";
              const alert=isLevelAlert(curLevel);
              return <div key={prod.id} style={{padding:"11px 0",borderBottom:"1px solid #F9FAFB"}}>
                <div style={{display:"flex",alignItems:"center",gap:9,marginBottom:9}}>
                  <span style={{fontSize:17,width:22,textAlign:"center"}}>{prod.icon}</span>
                  <div style={{flex:1}}>
                    <div style={{fontWeight:600,fontSize:13,color:"#111827"}}>{prod.label}</div>
                    <div style={{fontSize:10,color:"#9CA3AF"}}>Produit multi-usage — alerte à ¼ contenant</div>
                  </div>
                  {alert&&<span style={{fontSize:11,fontWeight:700,color:"#EF4444"}}>🔴 À renouveler</span>}
                </div>
                <div style={{display:"flex",gap:6,paddingLeft:31}}>
                  {LEVELS_PPL.map(lv=>{
                    const fill=LEVEL_FILLS[lv];
                    const lAlert=isLevelAlert(lv);
                    const color=lAlert?"#EF4444":fill>=75?"#10B981":"#F59E0B";
                    const sel=curLevel===lv;
                    return <button key={lv} onClick={()=>setQty(p=>({...p,[prod.id]:lv}))} style={{display:"flex",flexDirection:"column",alignItems:"center",gap:4,background:"none",border:"none",cursor:"pointer",padding:3,opacity:sel?1:0.45,transform:sel?"scale(1.12)":"scale(1)",transition:"all 0.12s"}}>
                      <div style={{width:24,height:40,borderRadius:4,border:`2px solid ${sel?color:"#D1D5DB"}`,background:"#F9FAFB",overflow:"hidden",position:"relative",boxShadow:sel?`0 2px 8px ${color}44`:"none"}}>
                        <div style={{position:"absolute",bottom:0,left:0,right:0,height:`${fill}%`,background:color,borderRadius:2}}/>
                        <div style={{position:"absolute",top:-3,left:"50%",transform:"translateX(-50%)",width:10,height:3,background:sel?color:"#D1D5DB",borderRadius:"2px 2px 0 0"}}/>
                      </div>
                      <span style={{fontSize:8,fontWeight:700,color:sel?color:"#9CA3AF",whiteSpace:"nowrap"}}>{LEVEL_LABELS[lv]}</span>
                    </button>;
                  })}
                </div>
              </div>;
            }
            const q=qty[prod.id]??0,ok=q>=prod.min,req=getReq(prod),enough=req===0||q>=req;
            return <div key={prod.id} style={{display:"flex",alignItems:"center",gap:9,padding:"9px 0",borderBottom:"1px solid #F9FAFB"}}>
              <span style={{fontSize:17,width:22,textAlign:"center"}}>{prod.icon}</span>
              <div style={{flex:1}}>
                <div style={{fontWeight:600,fontSize:13,color:"#111827"}}>{prod.label}</div>
                <div style={{fontSize:10,color:"#9CA3AF"}}>Min {prod.min} {prod.unite}{req>0&&<span style={{color:enough?"#10B981":"#F59E0B",marginLeft:6}}>· Besoin: {req}</span>}</div>
              </div>
              <div style={{display:"flex",alignItems:"center",gap:5}}>
                <button onClick={()=>setQty(p=>({...p,[prod.id]:Math.max(0,(p[prod.id]??0)-1)}))} style={{width:26,height:26,borderRadius:7,border:"1px solid #E5EDF2",background:"#F9FAFB",fontSize:15,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center",fontWeight:700}}>−</button>
                <input type="number" value={String(q)} onChange={e=>setQty(p=>({...p,[prod.id]:Math.max(0,parseInt(e.target.value)||0)}))} style={{width:42,textAlign:"center",fontSize:14,fontWeight:700,border:`1px solid ${ok?"#A7F3D0":"#FCA5A5"}`,borderRadius:7,padding:"3px 0",color:ok?"#10B981":"#EF4444",background:"#fff",outline:"none"}}/>
                <button onClick={()=>setQty(p=>({...p,[prod.id]:(p[prod.id]??0)+1}))} style={{width:26,height:26,borderRadius:7,border:"1px solid #E5EDF2",background:"#F9FAFB",fontSize:15,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center",fontWeight:700}}>+</button>
              </div>
              <span style={{fontSize:16}}>{ok?"✅":"🔴"}</span>
            </div>;
          })}
        </div>
      ))}
      <div style={{display:"flex",gap:8}}><Btn v="outline" onClick={()=>setStep("maintenance")}>← Retour</Btn><Btn onClick={()=>setStep("recap")} sx={{flex:1}}>Récapitulatif →</Btn></div>
    </div>}

    {step==="recap"&&<div>
      <div style={{fontWeight:700,fontSize:16,marginBottom:2}}>✅ Récapitulatif PPL</div>
      <div style={{fontSize:13,color:"#6B7280",marginBottom:12}}>{prop?.name} · {ppl.date}</div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:8,marginBottom:12}}>
        {[{icon:"🧹",val:score!=null?`${score}/100`:"—",label:"Propreté",c:score!=null?sc(score):"#9CA3AF"},{icon:"📍",val:Object.keys(vigPhotos).length>0?`${Object.values(vigPhotos).filter(v=>v.result?.ok).length}/${Object.keys(vigPhotos).length}`:"-",label:"Vigilance",c:"#10B981"},{icon:"🔧",val:maintItems.filter(m=>m.desc.trim()).length>0?`${maintItems.filter(m=>m.desc.trim()).length} pb`:"RAS",label:"Maintenance",c:maintItems.filter(m=>m.desc.trim()).length>0?"#F59E0B":"#10B981"}].map((s,i)=>(
          <div key={i} style={{background:"#fff",border:"1px solid #E5EDF2",borderRadius:11,padding:"11px 13px",textAlign:"center"}}>
            <div style={{fontSize:20,marginBottom:4}}>{s.icon}</div>
            <div style={{fontSize:16,fontWeight:800,color:s.c,lineHeight:1,marginBottom:2}}>{s.val}</div>
            <div style={{fontSize:10,color:"#9CA3AF",textTransform:"uppercase",letterSpacing:"0.07em"}}>{s.label}</div>
          </div>
        ))}
      </div>
      <div style={{...card,borderLeft:`4px solid ${(produits||[]).some(p=>p.type==="level"?isLevelAlert(qty[p.id]||"full"):(qty[p.id]??0)<p.min)?"#F59E0B":"#10B981"}`}}>
        <div style={{fontWeight:600,fontSize:13,color:"#111827",marginBottom:7}}>📦 Stocks — transmis au gestionnaire</div>
        <div style={{display:"flex",gap:6,flexWrap:"wrap"}}>{(produits||[]).map(p=>{
          if(p.type==="level"){const lv=qty[p.id]||"full";const alert=isLevelAlert(lv);return <div key={p.id} style={{background:alert?"#FFF1F1":"#ECFDF5",border:`1px solid ${alert?"#FCA5A5":"#6EE7B7"}`,borderRadius:8,padding:"4px 9px",fontSize:12,color:alert?"#991B1B":"#065F46"}}>{p.icon} {LEVEL_LABELS[lv]}</div>;}
          const q=qty[p.id]??0,ok=q>=p.min;return <div key={p.id} style={{background:ok?"#ECFDF5":"#FFF1F1",border:`1px solid ${ok?"#6EE7B7":"#FCA5A5"}`,borderRadius:8,padding:"4px 9px",fontSize:12,color:ok?"#065F46":"#991B1B"}}>{p.icon} {q}{!ok&&` ⚠`}</div>;
        })}</div>
      </div>
      {maintItems.filter(m=>m.desc.trim()).length>0&&<div style={{...card,borderLeft:"4px solid #F59E0B"}}>
        <div style={{fontWeight:600,fontSize:13,marginBottom:6}}>🔧 Problèmes signalés</div>
        {maintItems.filter(m=>m.desc.trim()).map((m,i)=><div key={i} style={{fontSize:13,color:"#374151",marginBottom:2}}>• {m.desc}</div>)}
      </div>}
      <div style={{display:"flex",gap:8,marginTop:8}}><Btn v="outline" onClick={()=>setStep("stock")}>← Retour</Btn><Btn onClick={submit} sx={{flex:1}}>✓ Valider et soumettre</Btn></div>
    </div>}
  </div>;
}

// ─── AIDE CHAT ────────────────────────────────────────────────────────────────
function AideChat({prestataire,ppls,properties,produits}) {
  const gp=id=>properties.find(p=>p.id===id);
  const [msgs,setMsgs]=useState([{role:"assistant",text:`Bonjour ${prestataire.name} 👋 Je suis votre assistant LOMMAX. Posez-moi vos questions sur vos logements, codes d'accès ou stocks.`}]);
  const [input,setInput]=useState("");
  const [loading,setLoading]=useState(false);
  const ref=useRef(null);
  useEffect(()=>ref.current?.scrollIntoView({behavior:"smooth"}),[msgs]);

  const sys=`Tu es l'assistant LOMMAX pour ${prestataire.name}.
LOGEMENTS: ${properties.filter(p=>prestataire.properties?.includes(p.id)).map(p=>`${p.name}: accès "${p.access}", ${p.duration}min`).join(" | ")}
PPL: ${ppls.filter(m=>m.prestataire===prestataire.id&&m.status!=="terminée").map(m=>`${gp(m.property)?.name} le ${m.date} à ${m.timeSlot}`).join("|")||"Aucun"}
Réponds en français, directement et de façon concise.`;

  const send=async()=>{
    if(!input.trim()||loading)return;
    const msg=input.trim();setInput("");setMsgs(p=>[...p,{role:"user",text:msg}]);setLoading(true);
    try{const t=await callClaude(sys,msg);setMsgs(p=>[...p,{role:"assistant",text:t}]);}
    catch{setMsgs(p=>[...p,{role:"assistant",text:"Erreur de connexion."}]);}
    setLoading(false);
  };

  return <div style={{display:"flex",flexDirection:"column",height:"calc(100vh - 140px)"}}>
    <div style={{fontWeight:700,fontSize:16,marginBottom:3}}>💬 Aide & Assistant</div>
    <div style={{fontSize:13,color:"#6B7280",marginBottom:10}}>Questions sur vos logements, accès, PPL</div>
    <div style={{display:"flex",gap:6,flexWrap:"wrap",marginBottom:10}}>
      {["Code d'accès ?","Ma prochaine mission ?","Points de vigilance ?"].map((q,i)=><button key={i} onClick={()=>setInput(q)} style={{padding:"5px 10px",borderRadius:20,border:`1px solid ${B.mid}`,background:B.light,color:B.dark,fontSize:12,cursor:"pointer",fontWeight:500}}>{q}</button>)}
    </div>
    <div style={{flex:1,overflowY:"auto",display:"flex",flexDirection:"column",gap:9,paddingBottom:7}}>
      {msgs.map((m,i)=><div key={i} style={{display:"flex",justifyContent:m.role==="user"?"flex-end":"flex-start"}}>
        <div style={{maxWidth:"82%",padding:"10px 14px",borderRadius:m.role==="user"?"14px 14px 4px 14px":"14px 14px 14px 4px",background:m.role==="user"?B.primary:"#fff",color:m.role==="user"?"#fff":"#374151",fontSize:13,lineHeight:1.6,border:m.role==="assistant"?"1px solid #E5EDF2":"none"}}>{m.text}</div>
      </div>)}
      {loading&&<div style={{display:"flex"}}><div style={{padding:"10px 14px",borderRadius:"14px 14px 14px 4px",background:"#fff",border:"1px solid #E5EDF2",fontSize:13,color:B.primary,fontStyle:"italic"}}>🤖 Réflexion...</div></div>}
      <div ref={ref}/>
    </div>
    <div style={{display:"flex",gap:7,paddingTop:9,borderTop:"1px solid #E5EDF2"}}>
      <input value={input} onChange={e=>setInput(e.target.value)} onKeyDown={e=>e.key==="Enter"&&!e.shiftKey&&send()} placeholder="Votre question..." style={{flex:1,background:"#F9FAFB",border:"1px solid #D1DAE3",borderRadius:10,padding:"10px 13px",fontSize:13,outline:"none"}}/>
      <Btn onClick={send} disabled={loading||!input.trim()} sx={{padding:"10px 14px"}}>↑</Btn>
    </div>
  </div>;
}

// ─── PRESTATAIRE APP ──────────────────────────────────────────────────────────
function PrestatairesApp({prestataire,onLogout,ppls,setPpls,documents,properties,messages,produits,stocks,onUpdateStocks}) {
  const gp=id=>properties.find(p=>p.id===id);
  const [tab,setTab]=useState("ppls");
  const [weekAnchor,setWeekAnchor]=useState(TODAY);
  const [detailPpl,setDetailPpl]=useState(null);
  const [activePpl,setActivePpl]=useState(null);
  const [pplDone,setPplDone]=useState(false);
  const [notif,setNotif]=useState(null);

  // PPLs attribués à moi (acceptés/terminés) + PPLs open pour mes logements
  const myPpls=ppls.filter(m=>m.prestataire===prestataire.id);
  const openPpls=ppls.filter(m=>m.status==="disponible"&&prestataire.properties?.includes(m.property));
  const pending=myPpls.filter(m=>m.status==="proposée"); // attribution directe
  const available=openPpls; // open pool
  const accepted=myPpls.filter(m=>m.status==="acceptée");
  const wDates=weekDates(weekAnchor);
  const myDocs=documents.filter(d=>d.prestataire===prestataire.id||d.prestataire==="all");
  const myMsgs=(messages||[]).filter(m=>m.targets==="all"||(Array.isArray(m.targets)&&m.targets.includes(prestataire.id)));
  const unread=myMsgs.filter(m=>!m.readBy?.includes(prestataire.id));
  const totalNew=pending.length+available.length;

  useEffect(()=>{
    if(totalNew>0){setNotif({count:totalNew,prop:gp((available[0]||pending[0])?.property)?.name});setTimeout(()=>setNotif(null),6000);}
  },[]);

  // Postuler à un PPL open (candidature → gestionnaire valide)
  const apply=(id)=>setPpls(p=>p.map(m=>m.id===id?{...m,candidates:[...new Set([...(m.candidates||[]),prestataire.id])]}:m));
  const cancelApply=(id)=>setPpls(p=>p.map(m=>m.id===id?{...m,candidates:(m.candidates||[]).filter(c=>c!==prestataire.id)}:m));
  // Refuser une attribution directe
  const respond=(id,action)=>setPpls(p=>p.map(m=>m.id===id?{...m,status:action==="accept"?"acceptée":"refusée"}:m));

  const TABS=[
    {id:"ppls",     icon:"📋",label:"PPL",     badge:totalNew},
    {id:"planning", icon:"📅",label:"Planning"},
    {id:"annonces", icon:"📣",label:"Annonces", badge:unread.length},
    {id:"documents",icon:"📁",label:"Docs"},
    {id:"aide",     icon:"💬",label:"Aide"},
  ];

  const navBtn=a=>({flex:1,padding:"9px 4px 11px",border:"none",background:"transparent",display:"flex",flexDirection:"column",alignItems:"center",gap:2,cursor:"pointer",borderTop:a?`3px solid ${B.primary}`:"3px solid transparent"});

  return <div style={{minHeight:"100vh",background:"#F4F7F9",fontFamily:"'DM Sans','Segoe UI',sans-serif",maxWidth:540,margin:"0 auto"}}>
    {notif&&<div style={{position:"fixed",top:0,left:"50%",transform:"translateX(-50%)",width:"min(540px,100vw)",zIndex:500,padding:"11px 14px",background:B.primary,color:"#fff",display:"flex",alignItems:"center",gap:9}}>
      <span style={{fontSize:18}}>📲</span>
      <div style={{flex:1}}><div style={{fontWeight:700,fontSize:13}}>{notif.count} nouveau{notif.count>1?"x":""} PPL proposé</div><div style={{fontSize:12,opacity:0.85}}>{notif.prop}</div></div>
      <button onClick={()=>setNotif(null)} style={{background:"none",border:"none",color:"#fff",fontSize:16,cursor:"pointer"}}>✕</button>
    </div>}

    <div style={{background:"#fff",borderBottom:"1px solid #E5EDF2",padding:"11px 14px",display:"flex",alignItems:"center",justifyContent:"space-between",position:"sticky",top:notif?52:0,zIndex:100}}>
      <div style={{display:"flex",alignItems:"center",gap:9}}>
        <Logo size="sm"/>
        <div style={{width:1,height:14,background:"#E5EDF2"}}/>
        <div><div style={{fontSize:13,fontWeight:600,color:"#374151"}}>{prestataire.name}</div><div style={{fontSize:10,color:"#9CA3AF"}}>{prestataire.role}</div></div>
      </div>
      <button onClick={onLogout} style={{background:"none",border:"none",color:"#9CA3AF",fontSize:11,cursor:"pointer"}}>Sortir</button>
    </div>

    <div style={{padding:"13px 13px 80px"}}>
      {tab==="ppls"&&(activePpl
        ?pplDone
          ?<div style={{textAlign:"center",paddingTop:50}}>
            <div style={{fontSize:50,marginBottom:12}}>✅</div>
            <div style={{fontSize:19,fontWeight:700,marginBottom:4}}>PPL soumis !</div>
            <div style={{fontSize:13,color:"#6B7280",marginBottom:22}}>Rapport et stocks transmis au gestionnaire.</div>
            <Btn onClick={()=>{setActivePpl(null);setPplDone(false);}} sx={{margin:"0 auto"}}>← Retour</Btn>
          </div>
          :<div>
            <button onClick={()=>setActivePpl(null)} style={{background:"none",border:"none",color:B.primary,fontSize:13,fontWeight:600,cursor:"pointer",marginBottom:14,display:"flex",alignItems:"center",gap:4}}>← {gp(activePpl.property)?.name}</button>
            <PPLForm ppl={activePpl} prestataire={prestataire} properties={properties} onComplete={()=>{setPpls(p=>p.map(m=>m.id===activePpl.id?{...m,status:"terminée"}:m));setPplDone(true);}} produits={produits} stocks={stocks} onUpdateStocks={onUpdateStocks}/>
          </div>
        :<div>
          {/* ── PPL OPEN (pool) ── */}
          {available.length>0&&<div style={{marginBottom:14}}>
            <div style={{fontSize:11,fontWeight:700,color:"#8B5CF6",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:9}}>⬡ Disponibles — {available.length} PPL à prendre en charge</div>
            {available.map(m=>{const prop=gp(m.property);return <div key={m.id} style={{...card,border:`2px solid #8B5CF644`,background:"#FDFBFF"}}>
              <div style={{display:"flex",justifyContent:"space-between",marginBottom:10}}>
                <div><div style={{fontWeight:700,fontSize:15,color:"#111827"}}>{prop?.name}</div><div style={{fontSize:11,color:"#9CA3AF"}}>{prop?.address}</div></div>
                <div style={{textAlign:"right",fontSize:12,color:"#6B7280"}}>{m.date}<br/>{m.timeSlot}</div>
              </div>
              <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:6,marginBottom:10}}>
                {[["⏱",`${prop?.duration} min`],["👥",`${m.guestCount||1} voyageur${(m.guestCount||1)>1?"s":""}`],["Check-in",m.nextCheckin||"—"],["Source",m.source==="lodgify"?"Lodgify":"Manuel"]].map(([l,v])=>(
                  <div key={l} style={{background:"#F9FAFB",borderRadius:8,padding:"6px 9px"}}><div style={{fontSize:10,color:"#9CA3AF",marginBottom:1}}>{l}</div><div style={{fontSize:12,fontWeight:600,color:"#111827"}}>{v}</div></div>
                ))}
              </div>
              <div style={{background:B.light,borderRadius:9,padding:"9px 11px",marginBottom:9}}>
                <div style={{fontSize:9,fontWeight:700,color:B.dark,textTransform:"uppercase",marginBottom:2}}>🔑 Accès</div>
                <div style={{fontSize:12,color:B.dark,fontWeight:500}}>{prop?.access}</div>
              </div>
              {m.vigilancePoints?.length>0&&<div style={{background:"#FFFBEB",border:"1px solid #FCD34D",borderRadius:9,padding:"9px 11px",marginBottom:9}}>
                <div style={{fontSize:9,fontWeight:700,color:"#92400E",textTransform:"uppercase",marginBottom:4}}>⚠ Points de vigilance</div>
                {m.vigilancePoints.map((p,i)=><div key={i} style={{display:"flex",gap:5,marginBottom:2}}><span style={{color:"#F59E0B",fontWeight:700}}>→</span><span style={{fontSize:12,color:"#374151"}}>{p}</span></div>)}
              </div>}
              {(() => {
                const alreadyApplied=(m.candidates||[]).includes(prestataire.id);
                return alreadyApplied
                  ?<div style={{display:"flex",gap:8,alignItems:"center"}}>
                    <div style={{flex:1,background:"#F0FDF4",border:"1px solid #6EE7B7",borderRadius:9,padding:"9px 14px",fontSize:13,color:"#065F46",fontWeight:600,textAlign:"center"}}>✓ Candidature envoyée — en attente de validation</div>
                    <Btn v="outline" onClick={()=>cancelApply(m.id)} sx={{padding:"9px 12px",fontSize:11,color:"#6B7280"}}>Annuler</Btn>
                  </div>
                  :<Btn onClick={()=>apply(m.id)} sx={{width:"100%",padding:"10px",background:"#8B5CF6",boxShadow:"0 3px 12px #8B5CF640"}}>👋 Je me propose pour ce PPL</Btn>;
              })()}
            </div>;})}
          </div>}

          {/* ── ATTRIBUTION DIRECTE ── */}
          {pending.length>0&&<div style={{marginBottom:14}}>
            <div style={{fontSize:11,fontWeight:700,color:"#F59E0B",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:9}}>📌 Attribué directement ({pending.length})</div>
            {pending.map(m=>{const prop=gp(m.property);return <div key={m.id} style={{...card,border:`2px solid ${B.primary}44`}}>
              <div style={{display:"flex",justifyContent:"space-between",marginBottom:10}}>
                <div><div style={{fontWeight:700,fontSize:15,color:"#111827"}}>{prop?.name}</div><div style={{fontSize:11,color:"#9CA3AF"}}>{prop?.address}</div></div>
                <div style={{textAlign:"right",fontSize:12,color:"#6B7280"}}>{m.date}<br/>{m.timeSlot}</div>
              </div>
              <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:6,marginBottom:10}}>
                {[["⏱",`${prop?.duration} min`],["👥",`${m.guestCount||1} voyageur${(m.guestCount||1)>1?"s":""}`],["Check-in",m.nextCheckin||"—"],["Durée estimée",`${prop?.duration} min`]].map(([l,v])=>(
                  <div key={l} style={{background:"#F9FAFB",borderRadius:8,padding:"6px 9px"}}><div style={{fontSize:10,color:"#9CA3AF",marginBottom:1}}>{l}</div><div style={{fontSize:12,fontWeight:600,color:"#111827"}}>{v}</div></div>
                ))}
              </div>
              <div style={{background:B.light,borderRadius:9,padding:"9px 11px",marginBottom:9}}>
                <div style={{fontSize:9,fontWeight:700,color:B.dark,textTransform:"uppercase",marginBottom:2}}>🔑 Accès</div>
                <div style={{fontSize:12,color:B.dark,fontWeight:500}}>{prop?.access}</div>
              </div>
              {m.vigilancePoints?.length>0&&<div style={{background:"#FFFBEB",border:"1px solid #FCD34D",borderRadius:9,padding:"9px 11px",marginBottom:9}}>
                <div style={{fontSize:9,fontWeight:700,color:"#92400E",textTransform:"uppercase",marginBottom:4}}>⚠ Points de vigilance</div>
                {m.vigilancePoints.map((p,i)=><div key={i} style={{display:"flex",gap:5,marginBottom:2}}><span style={{color:"#F59E0B",fontWeight:700}}>→</span><span style={{fontSize:12,color:"#374151"}}>{p}</span></div>)}
              </div>}
              <div style={{display:"flex",gap:7}}>
                <Btn v="danger" onClick={()=>respond(m.id,"decline")} sx={{flex:1,padding:"9px"}}>✕ Refuser</Btn>
                <Btn onClick={()=>respond(m.id,"accept")} sx={{flex:2,padding:"9px"}}>✓ Accepter</Btn>
              </div>
            </div>;})}
          </div>}

          {/* ── ACCEPTÉS ── */}
          {accepted.length>0&&<div style={{marginTop:available.length>0||pending.length>0?0:0}}>
            <div style={{fontSize:11,fontWeight:700,color:"#10B981",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:9}}>✓ Mes PPL ({accepted.length})</div>
            {accepted.map(m=>{const prop=gp(m.property),isToday=m.date===TODAY;return <div key={m.id} style={{...card,borderLeft:`4px solid ${isToday?"#10B981":B.primary}`}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:9}}>
                <div><div style={{fontWeight:700,fontSize:14}}>{prop?.name}</div><div style={{fontSize:12,color:"#9CA3AF"}}>{m.date} · {m.timeSlot} · {prop?.duration}min · {m.guestCount||1} voy.</div></div>
                {isToday&&<Bdg color="#10B981" bg="#ECFDF5">Aujourd'hui</Bdg>}
              </div>
              <div style={{display:"flex",gap:7}}>
                <Btn v="outline" onClick={()=>setDetailPpl(m)} sx={{flex:1,padding:"7px",fontSize:12}}>Détails</Btn>
                <Btn v="green" onClick={()=>{setActivePpl(m);setPplDone(false);}} sx={{flex:2,padding:"7px",fontSize:12}}>📋 Remplir le PPL</Btn>
              </div>
            </div>;})}
          </div>}

          {available.length===0&&pending.length===0&&accepted.length===0&&<div style={{...card,textAlign:"center",padding:38}}><div style={{fontSize:38}}>📋</div><div style={{fontWeight:600,color:"#374151",marginTop:8}}>Aucun PPL en cours</div></div>}
        </div>
      )}

      {tab==="planning"&&<div>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:14}}>
          <div style={{fontWeight:700,fontSize:16}}>Planning</div>
          <div style={{display:"flex",gap:7}}>
            <Btn v="outline" onClick={()=>{const d=new Date(weekAnchor);d.setDate(d.getDate()-7);setWeekAnchor(d.toISOString().split("T")[0]);}} sx={{padding:"5px 9px"}}>←</Btn>
            <Btn v="outline" onClick={()=>{const d=new Date(weekAnchor);d.setDate(d.getDate()+7);setWeekAnchor(d.toISOString().split("T")[0]);}} sx={{padding:"5px 9px"}}>→</Btn>
          </div>
        </div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(7,1fr)",gap:4,marginBottom:4}}>
          {wDates.map((date,i)=>{const isToday=date===TODAY;return <div key={date} style={{textAlign:"center"}}>
            <div style={{fontSize:9,color:"#9CA3AF",textTransform:"uppercase"}}>{WDAYS[i]}</div>
            <div style={{fontSize:12,fontWeight:isToday?700:400,color:isToday?B.primary:"#374151",width:24,height:24,borderRadius:"50%",background:isToday?B.light:"transparent",display:"flex",alignItems:"center",justifyContent:"center",margin:"2px auto"}}>{new Date(date).getDate()}</div>
          </div>;})}
        </div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(7,1fr)",gap:4}}>
          {wDates.map(date=>{
            const dm=[...myPpls,...openPpls].filter((m,i,a)=>a.findIndex(x=>x.id===m.id)===i).filter(m=>m.date===date&&m.status!=="refusée");
            return <div key={date} style={{minHeight:75,background:date===TODAY?"#F0F9FF":"#fff",border:`1px solid ${date===TODAY?B.mid:"#E5EDF2"}`,borderRadius:9,padding:"4px"}}>
              {dm.length===0?<div style={{fontSize:9,color:"#E5E7EB",textAlign:"center",marginTop:12}}>—</div>
              :dm.map(m=><div key={m.id} style={{background:m.status==="terminée"?"#F3F4F6":B.light,borderRadius:5,padding:"3px 4px",marginBottom:2,cursor:"pointer",borderLeft:`3px solid ${m.status==="terminée"?"#9CA3AF":B.primary}`}} onClick={()=>setDetailPpl(m)}>
                <div style={{fontSize:9,fontWeight:700,color:m.status==="terminée"?"#9CA3AF":B.dark}}>{gp(m.property)?.name?.split(" ")[0]}</div>
                <div style={{fontSize:8,color:"#9CA3AF"}}>{m.timeSlot}</div>
              </div>)}
            </div>;
          })}
        </div>
      </div>}

      {tab==="annonces"&&<div>
        <div style={{fontWeight:700,fontSize:16,marginBottom:4}}>📣 Annonces LOMMAX</div>
        {unread.length>0&&<div style={{background:"#EFF6FF",border:"1px solid #93C5FD",borderRadius:10,padding:"9px 13px",marginBottom:11,fontSize:13,color:"#1E40AF",fontWeight:500}}>🔵 {unread.length} nouveau{unread.length>1?"x":""} message{unread.length>1?"s":""}</div>}
        {myMsgs.length===0?<div style={{...card,textAlign:"center",padding:36}}><div style={{fontSize:36}}>📣</div><div style={{fontWeight:600,color:"#374151",marginTop:8}}>Aucune annonce</div></div>
        :[...myMsgs].sort((a,b)=>(b.pinned?1:0)-(a.pinned?1:0)||b.date.localeCompare(a.date)).map(msg=>(
          <AnnonceCard key={msg.id} msg={msg} isNew={!msg.readBy?.includes(prestataire.id)}/>
        ))}
        <div style={{fontSize:11,color:"#9CA3AF",textAlign:"center",marginTop:7,fontStyle:"italic"}}>Seuls vous et le gestionnaire avez accès à ces messages.</div>
      </div>}

      {tab==="documents"&&<div>
        <div style={{fontWeight:700,fontSize:16,marginBottom:4}}>📁 Mes documents</div>
        {myDocs.length===0?<div style={{...card,textAlign:"center",padding:36}}><div style={{fontSize:36}}>📁</div><div style={{fontWeight:600,color:"#374151",marginTop:8}}>Aucun document</div></div>
        :myDocs.map(doc=><div key={doc.id} style={{...card,display:"flex",alignItems:"center",gap:11}}>
          <div style={{width:36,height:36,borderRadius:9,background:doc.type?.includes("pdf")?"#FFF1F1":"#EFF6FF",display:"flex",alignItems:"center",justifyContent:"center",fontSize:16}}>{doc.type?.includes("pdf")?"📄":"📝"}</div>
          <div style={{flex:1}}><div style={{fontSize:13,fontWeight:600,color:"#111827"}}>{doc.name}</div><div style={{fontSize:11,color:"#9CA3AF"}}>{(doc.size/1024).toFixed(0)} Ko</div></div>
          <a href={doc.url} target="_blank" rel="noreferrer" style={{padding:"6px 12px",borderRadius:8,background:B.light,border:`1px solid ${B.mid}`,color:B.dark,fontSize:13,fontWeight:500,textDecoration:"none"}}>👁</a>
        </div>)}
      </div>}

      {tab==="aide"&&<AideChat prestataire={prestataire} ppls={ppls} properties={properties} produits={produits}/>}
    </div>

    <div style={{position:"fixed",bottom:0,left:"50%",transform:"translateX(-50%)",width:"min(540px,100vw)",background:"#fff",borderTop:"1px solid #E5EDF2",display:"flex",zIndex:100}}>
      {TABS.map(t=>(
        <button key={t.id} style={navBtn(tab===t.id)} onClick={()=>{setTab(t.id);setActivePpl(null);}}>
          <div style={{position:"relative",fontSize:19}}>
            {t.icon}
            {(t.badge||0)>0&&<span style={{position:"absolute",top:-4,right:-6,background:"#EF4444",color:"#fff",fontSize:8,fontWeight:700,borderRadius:8,padding:"1px 4px",lineHeight:"14px"}}>{t.badge}</span>}
          </div>
          <span style={{fontSize:10,fontWeight:tab===t.id?700:400,color:tab===t.id?B.primary:"#9CA3AF"}}>{t.label}</span>
        </button>
      ))}
    </div>

    {detailPpl&&<Modal onClose={()=>setDetailPpl(null)} w="min(420px,94vw)">
      <div style={{display:"flex",justifyContent:"space-between",marginBottom:12}}><div style={{fontWeight:700,fontSize:15}}>{gp(detailPpl.property)?.name}</div><Btn v="outline" onClick={()=>setDetailPpl(null)} sx={{padding:"4px 8px",fontSize:11}}>✕</Btn></div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:7,marginBottom:11}}>
        {[["📅",`${detailPpl.date} ${detailPpl.timeSlot}`],["⏱",`${gp(detailPpl.property)?.duration} min`],["👥",`${detailPpl.guestCount||1} voyageur${(detailPpl.guestCount||1)>1?"s":""}`],["Check-in",detailPpl.nextCheckin]].map(([l,v])=>(
          <div key={l} style={{background:"#F9FAFB",borderRadius:8,padding:"7px 10px"}}><div style={{fontSize:10,color:"#9CA3AF",marginBottom:1}}>{l}</div><div style={{fontSize:12,fontWeight:600}}>{v}</div></div>
        ))}
      </div>
      <div style={{background:B.light,borderRadius:9,padding:"10px 12px",marginBottom:9}}><div style={{fontSize:9,fontWeight:700,color:B.dark,textTransform:"uppercase",marginBottom:2}}>🔑 Accès</div><div style={{fontSize:13,color:B.dark,fontWeight:500}}>{gp(detailPpl.property)?.access}</div></div>
      {detailPpl.vigilancePoints?.length>0&&<div style={{background:"#FFFBEB",border:"1px solid #FCD34D",borderRadius:9,padding:"10px 12px"}}>
        <div style={{fontSize:9,fontWeight:700,color:"#92400E",textTransform:"uppercase",marginBottom:4}}>⚠ Points de vigilance</div>
        {detailPpl.vigilancePoints.map((p,i)=><div key={i} style={{display:"flex",gap:6,marginBottom:2}}><span style={{color:"#F59E0B",fontWeight:700}}>→</span><span style={{fontSize:12,color:"#374151"}}>{p}</span></div>)}
      </div>}
    </Modal>}
  </div>;
}

// ─── ROOT ─────────────────────────────────────────────────────────────────────
export default function App() {
  const [role,setRole]=useState(null);
  const [prestataire,setPrestataire]=useState(null);
  const [ppls,setPpls]=useState(INIT_PPLS);
  const [documents,setDocuments]=useState([]);
  const [properties,setProperties]=useState(INIT_PROPS);
  const [prestataires,setPrestataires]=useState(INIT_PRESTATAIRES);
  const [messages,setMessages]=useState(INIT_MESSAGES);
  const [produits,setProduits]=useState(INIT_PRODUITS);
  const [stocks,setStocks]=useState(INIT_STOCKS);
  const updateStocks=(propId,update)=>setStocks(prev=>({...prev,[propId]:{...(prev[propId]||{}),...update}}));

  if(!role) return <LoginScreen onLogin={(r,p)=>{setRole(r);setPrestataire(p);}} prestataires={prestataires}/>;
  if(role==="manager") return <ManagerApp onLogout={()=>setRole(null)} ppls={ppls} setPpls={setPpls} documents={documents} setDocuments={setDocuments} properties={properties} setProperties={setProperties} prestataires={prestataires} setPrestataires={setPrestataires} messages={messages} setMessages={setMessages} produits={produits} setProduits={setProduits} stocks={stocks} setStocks={setStocks}/>;
  return <PrestatairesApp prestataire={prestataire} onLogout={()=>setRole(null)} ppls={ppls} setPpls={setPpls} documents={documents} properties={properties} messages={messages} produits={produits} stocks={stocks} onUpdateStocks={updateStocks}/>;
}
