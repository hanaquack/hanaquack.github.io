<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"/>
<title>EmbodyYou</title>
<meta name="description" content="A supportive, female-focused fitness community app"/>
<meta name="theme-color" content="#FF7BB6"/>
<link rel="preconnect" href="https://fonts.googleapis.com"/>
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,400;0,9..40,600;0,9..40,700;0,9..40,800;1,9..40,800&family=Playfair+Display:ital,wght@1,700;1,800&display=swap" rel="stylesheet"/>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  html, body, #root { height: 100%; width: 100%; overflow: hidden; }
  body { font-family: 'DM Sans', 'Nunito', system-ui, sans-serif; -webkit-font-smoothing: antialiased; background: #111118; }
  input, select, button, textarea { font-family: inherit; }
  input:focus, select:focus { outline: none; border-color: #FF7BB6 !important; }
  button { -webkit-tap-highlight-color: transparent; }
  ::-webkit-scrollbar { width: 0; height: 0; }
  @keyframes loadBar { from { width: 0% } to { width: 100% } }
  @keyframes fadeIn { from { opacity: 0; transform: translateY(8px) } to { opacity: 1; transform: translateY(0) } }
  @keyframes pulse { 0%,100% { transform: scale(1) } 50% { transform: scale(1.05) } }
  .fade-in { animation: fadeIn 0.4s ease both; }
  .pulse { animation: pulse 2s ease infinite; }
  select { -webkit-appearance: none; appearance: none; background-image: url("data:image/svg+xml,%3Csvg width='12' height='8' viewBox='0 0 12 8' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M1 1L6 6L11 1' stroke='%23868DA6' stroke-width='2' stroke-linecap='round'/%3E%3C/svg%3E"); background-repeat: no-repeat; background-position: right 16px center; }
</style>
</head>
<body>
<div id="root"></div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.9/babel.min.js"></script>
<script type="text/babel">
const { useState, useEffect, useRef, useCallback } = React;

// ─── Storage ─────────────────────────────────────────
const STORE = "embody-you-v2";
const load = () => { try { return JSON.parse(localStorage.getItem(STORE)); } catch { return null; } };
const save = (d) => { try { localStorage.setItem(STORE, JSON.stringify(d)); } catch {} };

const BLANK = {
  name:"", email:"", password:"", age:28, gender:"", height:"", weight:"",
  healthNotes:"", onboarded:false, loggedIn:false,
  buddyProfile:{ location:"", fitnessGoal:"", workoutType:"", age:28 },
  encouragementStyle:"", mood:"", tokens:0, badges:[],
  journalEntries:[], matchedBuddies:[], messages:{},
};

// ─── Colors ──────────────────────────────────────────
const C = {
  pink:"#FF7BB6", pinkLight:"#FFD6EC", pinkDark:"#FF2DA4", pinkBg:"#FFF0F7",
  orange:"#FF9900", orangeLight:"#FFF6E6", orangeAccent:"#FFA51F", orangeWarm:"#FFB833",
  green:"#76C667", greenLight:"#E8F5E3",
  white:"#FFFFFF", gray50:"#FAFBFD", gray100:"#F6F8FC", gray200:"#EBEFF6",
  gray500:"#868DA6", black:"#1A1A2E", red:"#FF4757",
};
const grad = (a,b) => `linear-gradient(180deg,${a} 0%,${b} 100%)`;

// ─── Data ────────────────────────────────────────────
const BUDDIES = [
  { id:1, name:"Catherine Shepard", age:21, location:"Dallas, TX", distance:"2.5 mi", about:"I'm pretty chill, love a good playlist, and I'm all about consistency over perfection. If you like running or light lifting, let's connect!", interests:["Running","Endurance","Mon & Wed"], emoji:"👩‍🦰", color:"#FF7BB6" },
  { id:2, name:"Mia Chen", age:24, location:"Austin, TX", distance:"4.1 mi", about:"Yoga enthusiast turned weightlifter. I believe in balance — intense sessions paired with mindful recovery.", interests:["Yoga","Weights","Tue & Thu"], emoji:"👩‍🦱", color:"#FFA51F" },
  { id:3, name:"Jordan Rivera", age:19, location:"Plano, TX", distance:"6.2 mi", about:"College athlete getting back into fitness. I love group workouts and need an accountability partner!", interests:["HIIT","Cardio","Weekends"], emoji:"👩", color:"#76C667" },
];
const WORKOUTS = [
  { name:"Bodyweight Abs", time:"12 min", level:"Intermediate", emoji:"🔥" },
  { name:"Dumbbell Legs", time:"30 min", level:"Beginner", emoji:"🏋️‍♀️" },
  { name:"HIIT Cardio", time:"20 min", level:"Advanced", emoji:"🏃‍♀️" },
  { name:"Yoga Flow", time:"25 min", level:"Beginner", emoji:"🧘" },
  { name:"Full Body Stretch", time:"15 min", level:"All Levels", emoji:"🤸" },
  { name:"Upper Body Power", time:"35 min", level:"Intermediate", emoji:"💪" },
];

// ─── UI Kit ──────────────────────────────────────────
function StatusBar({light}) {
  const c = light ? "rgba(255,255,255,0.85)" : C.black;
  return React.createElement("div", {style:{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"12px 24px 4px",fontSize:13,fontWeight:600,color:c,flexShrink:0,minHeight:32}},
    React.createElement("span",null,"9:41"),
    React.createElement("div",{style:{display:"flex",gap:4,alignItems:"center"}},
      React.createElement("div",{style:{width:22,height:11,border:`1.5px solid ${c}`,borderRadius:3,position:"relative"}},
        React.createElement("div",{style:{position:"absolute",left:2,top:2,bottom:2,width:"65%",background:c,borderRadius:1}})
      )
    )
  );
}

function Btn({children, bg=C.orange, color="#fff", outline, full=true, disabled, onClick, s={}}) {
  return React.createElement("button",{disabled,onClick,style:{
    width:full?"100%":"auto",padding:"15px 24px",borderRadius:80,
    border:outline?`2px solid ${bg}`:"none",
    background:disabled?C.gray200:outline?"transparent":bg,
    color:disabled?C.gray500:outline?bg:color,
    fontSize:15,fontWeight:600,cursor:disabled?"not-allowed":"pointer",
    fontFamily:"inherit",transition:"all 0.2s",
    display:"flex",alignItems:"center",justifyContent:"center",gap:6,
    opacity:disabled?0.6:1,...s
  }},children);
}

function Input({label,value,onChange,placeholder,type="text",icon}) {
  return React.createElement("div",{style:{width:"100%"}},
    label && React.createElement("label",{style:{fontSize:14,fontWeight:600,marginBottom:6,display:"block",color:C.black}},label),
    React.createElement("div",{style:{display:"flex",alignItems:"center",background:C.white,border:`1.5px solid ${C.gray200}`,borderRadius:80,padding:"0 18px",transition:"border-color 0.2s"}},
      React.createElement("input",{type,value,onChange:e=>onChange(e.target.value),placeholder,style:{flex:1,border:"none",outline:"none",padding:"14px 0",fontSize:15,fontFamily:"inherit",background:"transparent",color:C.black,width:"100%"}}),
      icon && React.createElement("span",{style:{fontSize:16,color:C.gray500,flexShrink:0}},icon)
    )
  );
}

function Select({label,value,onChange,options,placeholder}) {
  return React.createElement("div",{style:{width:"100%"}},
    label && React.createElement("label",{style:{fontSize:14,fontWeight:600,marginBottom:6,display:"block"}},label),
    React.createElement("select",{value,onChange:e=>onChange(e.target.value),style:{
      width:"100%",padding:"14px 18px",borderRadius:16,border:`1.5px solid ${C.gray200}`,
      background:C.white,fontSize:15,fontFamily:"inherit",color:value?C.black:C.gray500,cursor:"pointer",outline:"none",
    }},
      React.createElement("option",{value:"",disabled:true},placeholder),
      options.map(o=>React.createElement("option",{key:o,value:o},o))
    )
  );
}

function SliderInput({label,value,onChange,min=18,max=55}) {
  return React.createElement("div",{style:{width:"100%"}},
    React.createElement("div",{style:{display:"flex",justifyContent:"space-between",marginBottom:8}},
      React.createElement("span",{style:{fontSize:14,fontWeight:600}},label),
      React.createElement("span",{style:{fontSize:14,fontWeight:700,color:C.orange}},value+(value>=55?"+":""))
    ),
    React.createElement("input",{type:"range",min,max,value,onChange:e=>onChange(Number(e.target.value)),
      style:{width:"100%",accentColor:C.orange,height:6,cursor:"pointer"}}),
    React.createElement("div",{style:{display:"flex",justifyContent:"space-between",fontSize:12,color:C.gray500,marginTop:4}},
      React.createElement("span",null,min),React.createElement("span",null,max+"+"))
  );
}

function NavBar({active,onNav}) {
  const items = [["🏠","Home",0],["🏋️","Workout",1],["🤝","Buddy",2],["⭐","Rewards",3],["👤","Profile",4]];
  return React.createElement("div",{style:{display:"flex",alignItems:"center",justifyContent:"space-around",padding:"10px 8px 20px",background:C.green,borderRadius:"22px 22px 0 0",flexShrink:0}},
    items.map(([ic,lb,idx])=>
      React.createElement("button",{key:idx,onClick:()=>onNav(idx),style:{
        display:"flex",flexDirection:"column",alignItems:"center",gap:2,
        background:"none",border:"none",cursor:"pointer",fontFamily:"inherit",
        opacity:active===idx?1:0.5,transform:active===idx?"scale(1.1)":"scale(1)",
        transition:"all 0.2s",padding:"4px 8px",
      }},
        React.createElement("span",{style:{fontSize:22}},ic),
        React.createElement("span",{style:{fontSize:9,color:C.white,fontWeight:700}},lb)
      )
    )
  );
}

function Toggle({on,onChange}) {
  return React.createElement("div",{onClick:onChange,style:{
    width:50,height:28,borderRadius:80,background:on?C.pinkDark:C.gray200,
    position:"relative",cursor:"pointer",transition:"background 0.2s",flexShrink:0,
  }},
    React.createElement("div",{style:{position:"absolute",top:2,left:on?24:2,width:24,height:24,borderRadius:"50%",background:C.white,transition:"left 0.2s",boxShadow:"0 1px 4px rgba(0,0,0,0.12)"}})
  );
}

function Dots({total,active,color=C.pinkDark}) {
  return React.createElement("div",{style:{display:"flex",gap:5,justifyContent:"center",padding:"10px 0",flexShrink:0}},
    Array.from({length:total}).map((_,i)=>
      React.createElement("div",{key:i,style:{width:i===active?22:8,height:7,borderRadius:200,background:i===active?color:`${color}30`,transition:"all 0.35s"}})
    )
  );
}

function Scroll({children,pb=0}) {
  return React.createElement("div",{style:{flex:1,overflowY:"auto",overflowX:"hidden",WebkitOverflowScrolling:"touch",paddingBottom:pb}},children);
}

// ═══════════════════════════════════════════════════════
// AUTH SCREENS
// ═══════════════════════════════════════════════════════

function SplashScreen({onNext}) {
  useEffect(()=>{ const t=setTimeout(onNext,2200); return ()=>clearTimeout(t); },[]);
  return React.createElement("div",{style:{flex:1,background:"linear-gradient(135deg,#FF7BB6,#FFB6D9 50%,#FF9ECE)",display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center"}},
    React.createElement("div",{className:"fade-in",style:{fontFamily:"'Playfair Display',serif",fontSize:52,fontWeight:800,color:C.white,fontStyle:"italic",letterSpacing:-3,lineHeight:1}},
      "embody",React.createElement("br"),React.createElement("span",{style:{fontWeight:400,fontSize:46}},"you.")
    ),
    React.createElement("div",{className:"fade-in",style:{marginTop:18,fontSize:10,color:"rgba(255,255,255,0.7)",fontWeight:600,letterSpacing:4,textTransform:"uppercase"}},"empower your every move"),
    React.createElement("div",{style:{marginTop:40,width:40,height:4,borderRadius:2,background:"rgba(255,255,255,0.4)",position:"relative",overflow:"hidden"}},
      React.createElement("div",{style:{position:"absolute",left:0,top:0,height:"100%",width:"100%",background:C.white,borderRadius:2,animation:"loadBar 2s linear"}})
    )
  );
}

function WelcomeScreen({onLogin,onSignup}) {
  return React.createElement("div",{style:{flex:1,background:grad(C.orangeLight,C.orangeAccent),display:"flex",flexDirection:"column"}},
    React.createElement(StatusBar),
    React.createElement("div",{style:{flex:1,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:"0 32px",textAlign:"center"}},
      React.createElement("div",{className:"pulse",style:{fontSize:72,marginBottom:12}},"🪽"),
      React.createElement("h1",{className:"fade-in",style:{fontSize:26,fontWeight:700,margin:"0 0 10px",letterSpacing:-0.5}},"Welcome to EmbodyYou"),
      React.createElement("p",{style:{fontSize:14,lineHeight:1.6,color:"#444",margin:"0 0 28px",maxWidth:300}},"A supportive, female-focused community to help you find gym partners, track your progress, and unlock your full potential."),
      React.createElement("div",{style:{width:"80%",display:"flex",flexDirection:"column",gap:10}},
        React.createElement(Btn,{onClick:onLogin},"Login"),
        React.createElement(Btn,{bg:C.orangeLight,color:C.orange,onClick:onSignup},"Create a New Account")
      )
    ),
    React.createElement("p",{style:{textAlign:"center",fontSize:11,color:"#777",padding:"16px 32px 24px",flexShrink:0}},"By continuing, you agree to our Terms and Privacy Policy.")
  );
}

function SignupScreen({user,setUser,onComplete,onBack}) {
  const [step,setStep]=useState(0);
  const [confirmPw,setConfirmPw]=useState("");
  const [error,setError]=useState("");

  const v0=()=>{
    if(!user.name.trim()){setError("Please enter your name");return false;}
    if(!user.email.includes("@")){setError("Valid email required");return false;}
    if(user.password.length<6){setError("Password: 6+ characters");return false;}
    setError("");return true;
  };

  return React.createElement("div",{style:{flex:1,background:grad(C.orangeLight,C.orangeAccent),display:"flex",flexDirection:"column"}},
    React.createElement(StatusBar),
    React.createElement("div",{style:{padding:"4px 20px 0",flexShrink:0}},
      React.createElement("button",{onClick:step===0?onBack:()=>setStep(0),style:{background:"rgba(255,255,255,0.3)",border:"none",borderRadius:200,width:38,height:38,fontSize:16,cursor:"pointer"}},"←")
    ),
    React.createElement(Scroll,{pb:24},
      React.createElement("div",{style:{padding:"12px 32px"}},
        React.createElement("h2",{style:{fontSize:26,fontWeight:700,margin:"0 0 4px"}},"Create Account"),
        React.createElement("p",{style:{fontSize:14,color:"#555",margin:"0 0 20px"}},step===0?"Enter your credentials":"Set up your password"),
        error&&React.createElement("div",{style:{background:`${C.red}15`,color:C.red,padding:"10px 14px",borderRadius:12,fontSize:13,marginBottom:14,fontWeight:500}},error),
        step===0?
          React.createElement("div",{style:{display:"flex",flexDirection:"column",gap:14}},
            React.createElement(Input,{label:"Full Name",value:user.name,onChange:v=>setUser({...user,name:v}),placeholder:"Your name"}),
            React.createElement(Input,{label:"Email",value:user.email,onChange:v=>setUser({...user,email:v}),placeholder:"you@email.com",type:"email"}),
            React.createElement(Input,{label:"Password",value:user.password,onChange:v=>setUser({...user,password:v}),placeholder:"At least 6 characters",type:"password",icon:"🔒"}),
            React.createElement(Btn,{onClick:()=>{if(v0())setStep(1)},s:{marginTop:8}},"Next →")
          ):
          React.createElement("div",{style:{display:"flex",flexDirection:"column",gap:14}},
            React.createElement(Input,{label:"Confirm Password",value:confirmPw,onChange:setConfirmPw,placeholder:"Re-enter password",type:"password"}),
            React.createElement("div",{style:{display:"flex",flexDirection:"column",gap:6}},
              [["6+ characters",user.password.length>=6],["Has a number",/\d/.test(user.password)],["Passwords match",confirmPw===user.password&&confirmPw.length>0]].map(([r,ok],i)=>
                React.createElement("div",{key:i,style:{display:"flex",alignItems:"center",gap:8}},
                  React.createElement("div",{style:{width:18,height:18,borderRadius:"50%",background:ok?"#4CAF50":C.gray200,display:"flex",alignItems:"center",justifyContent:"center",fontSize:10,color:"#fff",transition:"background 0.2s"}},ok?"✓":""),
                  React.createElement("span",{style:{fontSize:13,color:ok?C.black:C.gray500}},r)
                )
              )
            ),
            React.createElement(Btn,{disabled:confirmPw!==user.password||user.password.length<6,onClick:()=>{setUser({...user,loggedIn:true});onComplete();}},"Create Account →")
          )
      )
    )
  );
}

function LoginScreen({user,setUser,onComplete,onBack}) {
  const [email,setEmail]=useState("");
  const [pw,setPw]=useState("");
  const [error,setError]=useState("");

  return React.createElement("div",{style:{flex:1,background:C.orangeLight,display:"flex",flexDirection:"column"}},
    React.createElement(StatusBar),
    React.createElement(Scroll,null,
      React.createElement("div",{style:{height:120,background:grad(C.gray200,C.gray100),borderRadius:"0 0 28px 28px",display:"flex",alignItems:"center",justifyContent:"center"}},
        React.createElement("span",{style:{fontSize:56,opacity:0.2}},"🏋️‍♀️")
      ),
      React.createElement("div",{style:{padding:"20px 32px 32px",background:C.orangeLight,borderRadius:"28px 28px 0 0",marginTop:-16}},
        React.createElement("div",{style:{textAlign:"center",marginBottom:20}},
          React.createElement("h2",{style:{fontSize:26,fontWeight:700,margin:"0 0 4px"}},"Sign in"),
          React.createElement("p",{style:{fontSize:14,color:"#666",margin:0}},"Log in and start your journey.")
        ),
        error&&React.createElement("div",{style:{background:`${C.red}15`,color:C.red,padding:"10px 14px",borderRadius:12,fontSize:13,marginBottom:14}},error),
        React.createElement("div",{style:{display:"flex",flexDirection:"column",gap:14}},
          React.createElement(Input,{label:"Email",value:email,onChange:setEmail,placeholder:"you@email.com",type:"email"}),
          React.createElement(Input,{label:"Password",value:pw,onChange:setPw,placeholder:"Your password",type:"password",icon:"🔒"}),
          React.createElement(Btn,{onClick:()=>{
            if(!email.includes("@")){setError("Valid email required");return;}
            if(pw.length<3){setError("Enter your password");return;}
            setUser({...user,email,loggedIn:true,name:user.name||email.split("@")[0]});
            onComplete();
          }},"Login →"),
          React.createElement("button",{onClick:onBack,style:{background:"none",border:"none",fontSize:14,cursor:"pointer",fontFamily:"inherit",color:C.gray500,padding:8}},"← Back")
        )
      )
    )
  );
}

// ═══ ONBOARDING ══════════════════════════════════════

function OnboardingFlow({user,setUser,onComplete}) {
  const [step,setStep]=useState(0);
  const total=5;
  const next=()=>{if(step<total-1)setStep(step+1);else{setUser({...user,onboarded:true});onComplete();}};

  const screens=[
    React.createElement("div",{key:0,style:{flex:1,background:grad(C.pinkLight,C.pink),display:"flex",flexDirection:"column",alignItems:"center",padding:"0 32px"}},
      React.createElement("div",{style:{width:160,height:220,borderRadius:80,background:"rgba(255,255,255,0.15)",margin:"20px 0 16px",display:"flex",alignItems:"center",justifyContent:"center",border:"2px solid rgba(255,255,255,0.2)"}},
        React.createElement("span",{style:{fontSize:72}},"🤳")
      ),
      React.createElement("h2",{style:{fontSize:26,fontWeight:700,margin:"0 0 8px",textAlign:"center"}},"Get Started"+(user.name?`, ${user.name}`:"")),
      React.createElement("p",{style:{fontSize:14,color:"#333",textAlign:"center",margin:"0 0 24px",lineHeight:1.5}},"We ask a few basics to tailor your support. You're always in control."),
      React.createElement("div",{style:{width:"75%",display:"flex",flexDirection:"column",gap:10}},
        React.createElement(Btn,{bg:C.pink,onClick:next},"Continue"),
        React.createElement(Btn,{bg:C.pinkLight,color:C.pink,onClick:onComplete},"Skip for now")
      )
    ),
    React.createElement("div",{key:1,style:{flex:1,background:grad(C.pinkLight,C.pink),display:"flex",flexDirection:"column",alignItems:"center",padding:"32px 32px 0"}},
      React.createElement("h2",{style:{fontSize:26,fontWeight:700,margin:"0 0 8px",textAlign:"center"}},"Select Your Age"),
      React.createElement("p",{style:{fontSize:14,color:"#333",textAlign:"center",margin:"0 0 40px"}},"Every age is the right age to start."),
      React.createElement(SliderInput,{label:"Age",value:user.age,onChange:v=>setUser({...user,age:v})}),
      React.createElement("div",{style:{marginTop:"auto",width:"100%",paddingBottom:8}},React.createElement(Btn,{bg:C.pink,onClick:next},"Continue"))
    ),
    React.createElement("div",{key:2,style:{flex:1,background:grad(C.pinkLight,C.pink),display:"flex",flexDirection:"column",alignItems:"center",padding:"32px 32px 0"}},
      React.createElement("h2",{style:{fontSize:26,fontWeight:700,margin:"0 0 8px",textAlign:"center"}},"Select Your Gender"),
      React.createElement("p",{style:{fontSize:14,color:"#333",textAlign:"center",margin:"0 0 28px",lineHeight:1.5}},"We use this only to tailor content."),
      React.createElement(Select,{label:"Gender",value:user.gender,onChange:v=>setUser({...user,gender:v}),options:["Female","Male","Non-binary","Prefer not to say"],placeholder:"Select gender"}),
      React.createElement("div",{style:{marginTop:"auto",width:"100%",paddingBottom:8}},React.createElement(Btn,{bg:C.pink,onClick:next},"Continue"))
    ),
    React.createElement(Scroll,{key:3},
      React.createElement("div",{style:{padding:"24px 32px"}},
        React.createElement("div",{style:{textAlign:"center",marginBottom:24}},
          React.createElement("h2",{style:{fontSize:26,fontWeight:700,margin:"0 0 4px"}},"A Few More Questions"),
          React.createElement("p",{style:{fontSize:14,color:"#555",margin:0}},"Almost there!")
        ),
        React.createElement("div",{style:{display:"flex",flexDirection:"column",gap:14}},
          React.createElement(Input,{label:"Height",value:user.height,onChange:v=>setUser({...user,height:v}),placeholder:"e.g. 5'4\""}),
          React.createElement(Input,{label:"Weight",value:user.weight,onChange:v=>setUser({...user,weight:v}),placeholder:"e.g. 130 lbs"}),
          React.createElement(Select,{label:"Health Notes",value:user.healthNotes,onChange:v=>setUser({...user,healthNotes:v}),options:["None","Asthma","Joint issues","Heart condition","Other"],placeholder:"Any health concerns?"}),
          React.createElement(Btn,{bg:C.pink,onClick:next,s:{marginTop:8}},"Continue")
        )
      )
    ),
    React.createElement("div",{key:4,style:{flex:1,display:"flex",flexDirection:"column",background:C.white}},
      React.createElement("div",{style:{flex:1,background:grad(C.gray100,C.gray200),borderRadius:"0 0 36px 36px",display:"flex",alignItems:"center",justifyContent:"center"}},
        React.createElement("span",{className:"pulse",style:{fontSize:110}},"🎉")
      ),
      React.createElement("div",{style:{padding:"28px 32px 20px",textAlign:"center"}},
        React.createElement("h2",{style:{fontSize:26,fontWeight:700,margin:"0 0 6px",color:C.pink}},"You're all set"+(user.name?`, ${user.name}`:"")),
        React.createElement("p",{style:{fontSize:14,color:C.pink,margin:"0 0 20px"}},"Let's explore what's waiting for you."),
        React.createElement(Btn,{bg:C.pink,onClick:next},"Let's Go! 🚀")
      )
    ),
  ];

  return React.createElement("div",{style:{flex:1,display:"flex",flexDirection:"column",background:step===3?C.pinkLight:step===4?C.white:"transparent"}},
    React.createElement(StatusBar),
    screens[step],
    React.createElement(Dots,{total,active:step}),
    React.createElement("div",{style:{height:16,flexShrink:0}})
  );
}

// ═══ TABS ════════════════════════════════════════════

function HomeTab({user}) {
  return React.createElement(Scroll,{pb:16},
    React.createElement("div",{style:{padding:"8px 16px"}},
      React.createElement("div",{style:{display:"flex",alignItems:"center",gap:10,marginBottom:16}},
        React.createElement("div",{style:{width:40,height:40,borderRadius:"50%",background:C.pinkDark,display:"flex",alignItems:"center",justifyContent:"center",fontSize:18}},"👩"),
        React.createElement("div",null,
          React.createElement("div",{style:{fontSize:16,fontWeight:700}},"Hey, "+(user.name||"there")+"! 👋"),
          React.createElement("div",{style:{fontSize:12,color:C.gray500}},"Ready to crush it today?")
        )
      ),
      React.createElement("div",{style:{display:"flex",justifyContent:"center",margin:"0 0 20px"}},
        React.createElement("div",{style:{width:160,height:160,borderRadius:"50%",background:`conic-gradient(${C.pink} 0% ${Math.min((user.tokens||0)*3,100)}%, ${C.pinkLight} ${Math.min((user.tokens||0)*3,100)}% 100%)`,display:"flex",alignItems:"center",justifyContent:"center"}},
          React.createElement("div",{style:{width:120,height:120,borderRadius:"50%",background:C.orangeLight,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center"}},
            React.createElement("span",{style:{fontSize:24,fontWeight:800}},Math.min((user.tokens||0)*3,100)+"%"),
            React.createElement("span",{style:{fontSize:10,color:"#666"}},"Goals Reached")
          )
        )
      ),
      React.createElement("h3",{style:{fontSize:17,fontWeight:800,margin:"0 0 10px"}},"Top Video Workouts"),
      React.createElement("div",{style:{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}},
        WORKOUTS.slice(0,4).map((w,i)=>
          React.createElement("div",{key:i,style:{background:C.gray100,borderRadius:18,overflow:"hidden"}},
            React.createElement("div",{style:{height:90,display:"flex",alignItems:"center",justifyContent:"center",background:i%2===0?grad(C.pinkLight,C.gray100):grad(C.orangeLight,C.gray100)}},
              React.createElement("span",{style:{fontSize:36}},w.emoji)
            ),
            React.createElement("div",{style:{padding:"6px 10px 10px"}},
              React.createElement("div",{style:{fontSize:13,fontWeight:600}},w.name),
              React.createElement("div",{style:{fontSize:11,color:C.gray500}},w.time+" · "+w.level)
            )
          )
        )
      )
    )
  );
}

function WorkoutTab() {
  const [active,setActive]=useState(null);
  const [timer,setTimer]=useState(0);
  const ref=useRef(null);

  const start=(i)=>{setActive(i);setTimer(0);ref.current=setInterval(()=>setTimer(t=>t+1),1000);};
  const stop=()=>{clearInterval(ref.current);setActive(null);setTimer(0);};
  const fmt=s=>`${String(Math.floor(s/60)).padStart(2,"0")}:${String(s%60).padStart(2,"0")}`;

  if(active!==null){
    const w=WORKOUTS[active];
    return React.createElement("div",{style:{flex:1,background:grad(C.orangeLight,C.orangeWarm),display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:"0 32px",textAlign:"center"}},
      React.createElement("span",{style:{fontSize:72,marginBottom:16}},w.emoji),
      React.createElement("h2",{style:{fontSize:22,fontWeight:700,margin:"0 0 4px"}},w.name),
      React.createElement("p",{style:{fontSize:13,color:"#555",margin:"0 0 24px"}},w.level+" · "+w.time),
      React.createElement("div",{style:{fontSize:48,fontWeight:800,fontVariantNumeric:"tabular-nums",margin:"0 0 32px",color:C.orange}},fmt(timer)),
      React.createElement(Btn,{bg:C.red,onClick:stop},"End Workout")
    );
  }

  return React.createElement(Scroll,{pb:16},
    React.createElement("div",{style:{padding:"8px 16px"}},
      React.createElement("h2",{style:{fontSize:20,fontWeight:700,margin:"0 0 4px"}},"Workouts 🏋️"),
      React.createElement("p",{style:{fontSize:13,color:C.gray500,margin:"0 0 16px"}},"Tap to start a workout timer"),
      React.createElement("div",{style:{display:"flex",flexDirection:"column",gap:10}},
        WORKOUTS.map((w,i)=>
          React.createElement("button",{key:i,onClick:()=>start(i),style:{display:"flex",alignItems:"center",gap:14,width:"100%",padding:"14px 16px",background:C.white,border:`1.5px solid ${C.gray200}`,borderRadius:18,cursor:"pointer",fontFamily:"inherit",textAlign:"left"}},
            React.createElement("div",{style:{width:48,height:48,borderRadius:14,background:i%2===0?C.pinkLight:C.orangeLight,display:"flex",alignItems:"center",justifyContent:"center",fontSize:24,flexShrink:0}},w.emoji),
            React.createElement("div",{style:{flex:1}},
              React.createElement("div",{style:{fontSize:15,fontWeight:600}},w.name),
              React.createElement("div",{style:{fontSize:12,color:C.gray500}},w.time+" · "+w.level)
            ),
            React.createElement("span",{style:{fontSize:18,color:C.orange}},"▶")
          )
        )
      )
    )
  );
}

function BuddyTab({user,setUser}) {
  const [view,setView]=useState("list");
  const [ci,setCi]=useState(0);
  const [mid,setMid]=useState(null);
  const [msg,setMsg]=useState("");
  const matched=user.matchedBuddies||[];
  const buddy=BUDDIES[ci];

  const swipeR=()=>{
    const nm=[...matched,buddy.id];
    setUser({...user,matchedBuddies:nm,messages:{...user.messages,[buddy.id]:[]}});
    setMid(buddy.id);setView("match");
  };
  const swipeL=()=>{if(ci<BUDDIES.length-1)setCi(ci+1);else setView("list");};
  const send=()=>{
    if(!msg.trim())return;
    const ms=[...(user.messages[mid]||[]),{text:msg,from:"me",time:new Date().toLocaleTimeString([],{hour:"2-digit",minute:"2-digit"})}];
    setUser({...user,messages:{...user.messages,[mid]:ms}});setMsg("");
  };

  if(view==="card"){
    return React.createElement("div",{style:{flex:1,display:"flex",flexDirection:"column"}},
      React.createElement("div",{style:{flex:1,margin:"4px 8px",borderRadius:24,overflow:"hidden",display:"flex",flexDirection:"column",boxShadow:"0 4px 20px rgba(0,0,0,0.1)"}},
        React.createElement("div",{style:{height:"48%",background:grad(buddy.color,buddy.color+"88"),display:"flex",alignItems:"center",justifyContent:"center",position:"relative",flexShrink:0}},
          React.createElement("span",{style:{fontSize:90}},buddy.emoji),
          React.createElement("div",{style:{position:"absolute",top:12,right:12,background:C.green,borderRadius:32,padding:"4px 10px",display:"flex",alignItems:"center",gap:3}},
            React.createElement("span",{style:{fontSize:9}},"📍"),
            React.createElement("span",{style:{fontSize:12,color:C.white,fontWeight:600}},buddy.distance)
          ),
          React.createElement("div",{style:{position:"absolute",bottom:16,left:16,color:C.white}},
            React.createElement("div",{style:{fontSize:20,fontWeight:800}},buddy.name),
            React.createElement("div",{style:{fontSize:10,letterSpacing:1.5,textTransform:"uppercase",opacity:0.85}},buddy.location)
          )
        ),
        React.createElement("div",{style:{flex:1,padding:"14px 16px",overflowY:"auto",background:C.white}},
          React.createElement("div",{style:{fontSize:11,color:C.gray500,marginBottom:3}},"About"),
          React.createElement("p",{style:{fontSize:13,lineHeight:1.6,margin:"0 0 12px",color:"#333"}},buddy.about),
          React.createElement("div",{style:{fontSize:11,color:C.gray500,marginBottom:6}},"Interests"),
          React.createElement("div",{style:{display:"flex",flexWrap:"wrap",gap:6}},
            [...buddy.interests,buddy.age+" yrs"].map((t,i)=>
              React.createElement("span",{key:i,style:{padding:"4px 10px",borderRadius:32,border:`1px solid ${C.orange}`,fontSize:11,background:C.white}},t)
            )
          )
        ),
        React.createElement("div",{style:{display:"flex",justifyContent:"center",gap:24,padding:"12px 0 16px",flexShrink:0,background:C.white}},
          React.createElement("button",{onClick:swipeL,style:{width:54,height:54,borderRadius:"50%",background:C.pink,border:"none",boxShadow:"0 4px 16px rgba(75,22,76,0.2)",display:"flex",alignItems:"center",justifyContent:"center",fontSize:20,color:C.white,cursor:"pointer"}},"✕"),
          React.createElement("button",{onClick:swipeR,style:{width:54,height:54,borderRadius:"50%",background:C.orangeAccent,border:"none",boxShadow:"0 4px 16px rgba(75,22,76,0.2)",display:"flex",alignItems:"center",justifyContent:"center",fontSize:20,cursor:"pointer"}},"💪")
        )
      )
    );
  }

  if(view==="match"){
    const mb=BUDDIES.find(b=>b.id===mid);
    return React.createElement("div",{style:{flex:1,background:grad(C.orangeLight,C.orangeWarm),display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:"0 32px",textAlign:"center"}},
      React.createElement("h2",{className:"fade-in",style:{fontSize:26,fontWeight:700,margin:"0 0 8px"}},"It's a Match! 🎉"),
      React.createElement("p",{style:{fontSize:14,color:"#333",margin:"0 0 28px"}},"You and "+mb.name+" can now chat!"),
      React.createElement("div",{style:{display:"flex",alignItems:"center",position:"relative",marginBottom:32}},
        React.createElement("div",{style:{width:110,height:110,borderRadius:"50%",background:grad(C.pinkLight,C.pink),display:"flex",alignItems:"center",justifyContent:"center",fontSize:48,border:`3px solid ${C.white}`,zIndex:2}},"👩"),
        React.createElement("div",{style:{width:110,height:110,borderRadius:"50%",background:grad(C.orangeLight,C.orangeWarm),display:"flex",alignItems:"center",justifyContent:"center",fontSize:48,border:`3px solid ${C.white}`,marginLeft:-20}},mb.emoji),
        React.createElement("div",{style:{position:"absolute",top:"50%",left:"50%",transform:"translate(-50%,-50%) rotate(15deg)",fontSize:32,zIndex:3}},"❤️")
      ),
      React.createElement("div",{style:{width:"100%",display:"flex",flexDirection:"column",gap:10}},
        React.createElement(Btn,{onClick:()=>setView("chat")},"Message "+mb.name),
        React.createElement(Btn,{bg:C.orangeLight,color:C.orange,onClick:()=>{setCi(Math.min(ci+1,BUDDIES.length-1));setView("card");}},"Keep Swiping")
      )
    );
  }

  if(view==="chat"){
    const mb=BUDDIES.find(b=>b.id===mid);
    const msgs=user.messages[mid]||[];
    return React.createElement("div",{style:{flex:1,display:"flex",flexDirection:"column"}},
      React.createElement("div",{style:{padding:"6px 12px 10px",borderBottom:`1px solid ${C.gray200}`,display:"flex",alignItems:"center",gap:10,flexShrink:0}},
        React.createElement("button",{onClick:()=>setView("list"),style:{background:"none",border:"none",fontSize:18,cursor:"pointer",padding:4}},"←"),
        React.createElement("span",{style:{flex:1,fontWeight:600,fontSize:15}},mb.name),
        React.createElement("div",{style:{width:34,height:34,borderRadius:"50%",background:mb.color,display:"flex",alignItems:"center",justifyContent:"center",fontSize:16}},mb.emoji)
      ),
      React.createElement("div",{style:{flex:1,overflowY:"auto",padding:12,display:"flex",flexDirection:"column",gap:8,justifyContent:msgs.length?"flex-end":"center"}},
        msgs.length===0&&React.createElement("p",{style:{textAlign:"center",color:C.gray500,fontSize:13}},"Say hi to "+mb.name+"! 👋"),
        msgs.map((m,i)=>
          React.createElement("div",{key:i,style:{display:"flex",justifyContent:m.from==="me"?"flex-end":"flex-start"}},
            React.createElement("div",{style:{maxWidth:"75%",background:m.from==="me"?C.orange:C.gray100,color:m.from==="me"?C.white:C.black,borderRadius:m.from==="me"?"18px 18px 4px 18px":"18px 18px 18px 4px",padding:"10px 14px",fontSize:14,lineHeight:1.5}},
              m.text,
              React.createElement("div",{style:{fontSize:10,opacity:0.6,marginTop:4,textAlign:"right"}},m.time)
            )
          )
        )
      ),
      React.createElement("div",{style:{padding:"10px 12px 16px",display:"flex",gap:8,flexShrink:0,background:C.gray50}},
        React.createElement("input",{value:msg,onChange:e=>setMsg(e.target.value),onKeyDown:e=>e.key==="Enter"&&send(),placeholder:"Type a message...",style:{flex:1,border:`1px solid ${C.gray200}`,borderRadius:24,padding:"10px 16px",fontSize:14,fontFamily:"inherit",outline:"none"}}),
        React.createElement("button",{onClick:send,style:{width:40,height:40,borderRadius:"50%",background:C.orange,border:"none",color:C.white,fontSize:16,cursor:"pointer",flexShrink:0}},"→")
      )
    );
  }

  return React.createElement(Scroll,null,
    React.createElement("div",{style:{padding:"8px 20px"}},
      React.createElement("h2",{style:{fontSize:22,fontWeight:700,margin:"0 0 4px"}},"Grow Together 🤝"),
      React.createElement("p",{style:{fontSize:13,color:C.gray500,margin:"0 0 20px"}},"Find your accountability partner"),
      React.createElement(Btn,{onClick:()=>{setCi(0);setView("card")},s:{marginBottom:20}},"Find a Partner →"),
      matched.length>0&&React.createElement(React.Fragment,null,
        React.createElement("h3",{style:{fontSize:15,fontWeight:700,margin:"0 0 10px"}},"Your Matches"),
        matched.map(id=>{
          const b=BUDDIES.find(x=>x.id===id);if(!b)return null;
          return React.createElement("button",{key:id,onClick:()=>{setMid(id);setView("chat");},style:{display:"flex",alignItems:"center",gap:12,width:"100%",padding:"12px 14px",background:C.white,border:`1px solid ${C.gray200}`,borderRadius:16,marginBottom:8,cursor:"pointer",fontFamily:"inherit",textAlign:"left"}},
            React.createElement("div",{style:{width:44,height:44,borderRadius:"50%",background:b.color,display:"flex",alignItems:"center",justifyContent:"center",fontSize:22,flexShrink:0}},b.emoji),
            React.createElement("div",{style:{flex:1}},
              React.createElement("div",{style:{fontSize:14,fontWeight:600}},b.name),
              React.createElement("div",{style:{fontSize:12,color:C.gray500}},(user.messages[id]||[]).length+" messages")
            ),
            React.createElement("span",{style:{fontSize:14,color:C.gray500}},"→")
          );
        })
      )
    )
  );
}

function RewardsTab({user,setUser}) {
  const [view,setView]=useState("main");
  const moods=[["😌","Calm"],["⚡","Energized"],["😔","Low"],["😰","Stressed"]];
  const badges=[["🏃","Consistency"],["💪","Strength"],["🫁","Endurance"],["🔥","Motivated"]];

  const checkin=(mood)=>{
    const nt=(user.tokens||0)+1;
    const nb=[...(user.badges||[])];
    if(nt>=3&&!nb.includes("Consistency"))nb.push("Consistency");
    if(nt>=5&&!nb.includes("Motivated"))nb.push("Motivated");
    if(nt>=7&&!nb.includes("Strength"))nb.push("Strength");
    if(nt>=10&&!nb.includes("Endurance"))nb.push("Endurance");
    setUser({...user,mood,tokens:nt,badges:nb});
    setView("token");
  };

  if(view==="checkin"){
    return React.createElement("div",{style:{flex:1,background:grad(C.pinkLight,C.pink),display:"flex",flexDirection:"column",padding:"20px 28px 0"}},
      React.createElement("button",{onClick:()=>setView("main"),style:{background:"none",border:"none",fontSize:16,cursor:"pointer",alignSelf:"flex-start",padding:4,marginBottom:8}},"← Back"),
      React.createElement("h2",{style:{fontSize:24,fontWeight:700,margin:"0 0 20px",textAlign:"center"}},"How Are You Feeling?"),
      React.createElement("div",{style:{display:"grid",gridTemplateColumns:"1fr 1fr",gap:12}},
        moods.map(([em,lb])=>
          React.createElement("button",{key:lb,onClick:()=>checkin(lb),style:{
            background:user.mood===lb?C.pinkDark:"rgba(255,255,255,0.35)",
            borderRadius:20,padding:"20px 10px",border:"none",cursor:"pointer",
            display:"flex",flexDirection:"column",alignItems:"center",gap:6,fontFamily:"inherit",
          }},
            React.createElement("span",{style:{fontSize:48}},em),
            React.createElement("span",{style:{fontSize:14,fontWeight:700,color:user.mood===lb?"#fff":"#333"}},lb)
          )
        )
      )
    );
  }

  if(view==="token"){
    return React.createElement("div",{style:{flex:1,background:grad(C.pinkLight,C.pink),display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:"0 32px",textAlign:"center"}},
      React.createElement("h2",{className:"fade-in",style:{fontSize:24,fontWeight:700,margin:"0 0 8px"}},"Growth Token Earned! 🌟"),
      React.createElement("span",{className:"pulse",style:{fontSize:120,margin:"12px 0"}},"⭐"),
      React.createElement("p",{style:{fontSize:16,fontWeight:600,margin:"0 0 8px"}},"You now have "+(user.tokens)+" token"+(user.tokens!==1?"s":"")),
      React.createElement("p",{style:{fontSize:13,color:"#444",margin:"0 0 28px"}},"Keep checking in to earn badges."),
      React.createElement(Btn,{bg:C.pink,onClick:()=>setView("main")},"Back to Rewards")
    );
  }

  return React.createElement(Scroll,{pb:16},
    React.createElement("div",{style:{padding:"8px 18px"}},
      React.createElement("h2",{style:{fontSize:20,fontWeight:700,margin:"0 0 16px"}},"Your Rewards ⭐"),
      React.createElement("div",{style:{background:grad(C.pink,C.pinkDark),borderRadius:20,padding:"20px 20px",marginBottom:16,display:"flex",alignItems:"center",justifyContent:"space-between"}},
        React.createElement("div",null,
          React.createElement("div",{style:{fontSize:13,color:"rgba(255,255,255,0.7)"}},"Growth Tokens"),
          React.createElement("div",{style:{fontSize:32,fontWeight:800,color:C.white}},user.tokens||0)
        ),
        React.createElement(Btn,{bg:"rgba(255,255,255,0.2)",color:C.white,full:false,onClick:()=>setView("checkin"),s:{padding:"10px 18px",fontSize:13}},"Check In →")
      ),
      user.mood&&React.createElement("div",{style:{background:C.white,border:`1px solid ${C.gray200}`,borderRadius:16,padding:"14px 16px",marginBottom:16}},
        React.createElement("div",{style:{fontSize:12,color:C.gray500,marginBottom:4}},"Current Mood"),
        React.createElement("div",{style:{fontSize:16,fontWeight:600}},(moods.find(m=>m[1]===user.mood)||[])[0]+" "+user.mood)
      ),
      React.createElement("h3",{style:{fontSize:16,fontWeight:700,margin:"0 0 10px"}},"Achievements"),
      React.createElement("div",{style:{display:"grid",gridTemplateColumns:"1fr 1fr 1fr 1fr",gap:8,marginBottom:20}},
        badges.map(([em,lb])=>{
          const earned=(user.badges||[]).includes(lb);
          return React.createElement("div",{key:lb,style:{display:"flex",flexDirection:"column",alignItems:"center",gap:4,opacity:earned?1:0.3}},
            React.createElement("div",{style:{width:50,height:50,borderRadius:"50%",background:earned?grad(C.pinkLight,C.pink):C.gray200,display:"flex",alignItems:"center",justifyContent:"center",fontSize:20,border:earned?`2px solid ${C.pinkDark}`:"none"}},em),
            React.createElement("span",{style:{fontSize:10,fontWeight:700,textAlign:"center"}},lb)
          );
        })
      ),
      React.createElement("h3",{style:{fontSize:16,fontWeight:700,margin:"0 0 10px"}},"Journal"),
      React.createElement(JournalSection,{user,setUser})
    )
  );
}

function JournalSection({user,setUser}) {
  const [entry,setEntry]=useState("");
  const entries=user.journalEntries||[];
  const add=()=>{
    if(!entry.trim())return;
    const ne=[{text:entry,date:new Date().toLocaleDateString("en-US",{weekday:"long",month:"long",day:"numeric"}),time:new Date().toLocaleTimeString([],{hour:"2-digit",minute:"2-digit"})},...entries];
    setUser({...user,journalEntries:ne});setEntry("");
  };
  return React.createElement("div",null,
    React.createElement("div",{style:{display:"flex",gap:8,marginBottom:12}},
      React.createElement("input",{value:entry,onChange:e=>setEntry(e.target.value),onKeyDown:e=>e.key==="Enter"&&add(),placeholder:"How was your workout?",style:{flex:1,border:`1.5px solid ${C.gray200}`,borderRadius:16,padding:"10px 14px",fontSize:13,fontFamily:"inherit",outline:"none"}}),
      React.createElement("button",{onClick:add,disabled:!entry.trim(),style:{background:C.pink,color:C.white,border:"none",borderRadius:16,padding:"0 16px",fontSize:13,fontWeight:600,cursor:entry.trim()?"pointer":"not-allowed",opacity:entry.trim()?1:0.5}},"Add")
    ),
    entries.length===0&&React.createElement("p",{style:{fontSize:13,color:C.gray500,textAlign:"center",padding:"20px 0"}},"No journal entries yet. Start writing!"),
    entries.map((e,i)=>
      React.createElement("div",{key:i,style:{background:C.white,border:`1.5px solid ${C.pink}33`,borderRadius:14,padding:"12px 14px",marginBottom:8}},
        React.createElement("p",{style:{fontSize:13,lineHeight:1.5,margin:"0 0 8px",color:"#333"}},e.text),
        React.createElement("div",{style:{display:"flex",justifyContent:"space-between",borderTop:`1px solid ${C.gray200}`,paddingTop:6}},
          React.createElement("span",{style:{fontSize:10,fontWeight:600,color:C.gray500}},e.date),
          React.createElement("span",{style:{fontSize:10,color:C.gray500}},e.time)
        )
      )
    )
  );
}

function ProfileTab({user,setUser,onLogout}) {
  return React.createElement(Scroll,null,
    React.createElement("div",{style:{padding:"8px 20px"}},
      React.createElement("div",{style:{display:"flex",alignItems:"center",gap:14,marginBottom:24}},
        React.createElement("div",{style:{width:64,height:64,borderRadius:"50%",background:grad(C.pinkLight,C.pink),display:"flex",alignItems:"center",justifyContent:"center",fontSize:28,border:`2px solid ${C.pinkDark}`}},"👩"),
        React.createElement("div",null,
          React.createElement("div",{style:{fontSize:20,fontWeight:700}},user.name||"User"),
          React.createElement("div",{style:{fontSize:13,color:C.gray500}},user.email)
        )
      ),
      React.createElement("div",{style:{display:"flex",flexDirection:"column",gap:12,marginBottom:24}},
        [["Age",user.age+(user.age>=55?"+":"")],["Gender",user.gender||"Not set"],["Height",user.height||"Not set"],["Weight",user.weight||"Not set"],["Tokens",(user.tokens||0)+" ⭐"],["Badges",(user.badges||[]).length>0?(user.badges||[]).join(", "):"None yet"],["Matches",(user.matchedBuddies||[]).length+" buddies"]].map(([l,v])=>
          React.createElement("div",{key:l,style:{display:"flex",justifyContent:"space-between",padding:"12px 14px",background:C.white,borderRadius:14,border:`1px solid ${C.gray200}`}},
            React.createElement("span",{style:{fontSize:14,color:C.gray500}},l),
            React.createElement("span",{style:{fontSize:14,fontWeight:600}},v)
          )
        )
      ),
      React.createElement(Btn,{bg:C.red,onClick:onLogout,s:{marginBottom:16}},"Log Out")
    )
  );
}

// ═══ MAIN APP ════════════════════════════════════════

function App() {
  const [user,setUser]=useState(BLANK);
  const [screen,setScreen]=useState("splash");
  const [tab,setTab]=useState(0);
  const [loaded,setLoaded]=useState(false);

  useEffect(()=>{
    const d=load();
    if(d){setUser({...BLANK,...d});if(d.loggedIn&&d.onboarded)setScreen("app");else if(d.loggedIn)setScreen("onboarding");else setScreen("splash");}
    setLoaded(true);
  },[]);

  useEffect(()=>{if(loaded)save(user);},[user,loaded]);

  const logout=()=>{setUser(BLANK);setScreen("splash");save(BLANK);};

  if(!loaded)return null;

  const tabBgs = [grad(C.orangeLight,C.orangeAccent), C.gray50, grad(C.orangeLight,C.orangeWarm), C.pinkBg, C.gray50];

  const renderScreen=()=>{
    switch(screen){
      case "splash": return React.createElement(SplashScreen,{onNext:()=>setScreen(user.loggedIn?(user.onboarded?"app":"onboarding"):"welcome")});
      case "welcome": return React.createElement(WelcomeScreen,{onLogin:()=>setScreen("login"),onSignup:()=>setScreen("signup")});
      case "signup": return React.createElement(SignupScreen,{user,setUser,onComplete:()=>setScreen("onboarding"),onBack:()=>setScreen("welcome")});
      case "login": return React.createElement(LoginScreen,{user,setUser,onComplete:()=>setScreen(user.onboarded?"app":"onboarding"),onBack:()=>setScreen("welcome")});
      case "onboarding": return React.createElement(OnboardingFlow,{user,setUser,onComplete:()=>{setUser(u=>({...u,onboarded:true}));setScreen("app");}});
      case "app": return React.createElement("div",{style:{flex:1,display:"flex",flexDirection:"column",background:tabBgs[tab]}},
        React.createElement(StatusBar),
        React.createElement("div",{style:{flex:1,display:"flex",flexDirection:"column",overflow:"hidden"}},
          tab===0&&React.createElement(HomeTab,{user}),
          tab===1&&React.createElement(WorkoutTab),
          tab===2&&React.createElement(BuddyTab,{user,setUser}),
          tab===3&&React.createElement(RewardsTab,{user,setUser}),
          tab===4&&React.createElement(ProfileTab,{user,setUser,onLogout:logout})
        ),
        React.createElement(NavBar,{active:tab,onNav:setTab})
      );
    }
  };

  return React.createElement("div",{style:{width:"100%",height:"100%",maxWidth:430,margin:"0 auto",background:C.black,display:"flex",flexDirection:"column",overflow:"hidden",position:"relative"}},
    renderScreen()
  );
}

ReactDOM.createRoot(document.getElementById("root")).render(React.createElement(App));
</script>
</body>
</html>
