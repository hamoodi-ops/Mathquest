import React, { useState, useRef, useEffect } from "react";
import { Canvas, useFrame } from "@react-three/fiber";
import { OrbitControls, Sphere, Box, Plane, Html } from "@react-three/drei";

// ---------- UTIL ----------
const rand = (n) => Math.floor(Math.random() * n);

function generateQuestion(grade) {
  let a = rand(grade * 10);
  let b = rand(grade * 10);
  return { q: `${a} + ${b}`, a: a + b };
}

// ---------- FLOATING DAMAGE TEXT ----------
function DamageText({ value, position }) {
  const ref = useRef();
  useFrame(() => {
    if (ref.current) ref.current.position.y += 0.02;
  });
  return (
    <Html position={position}>
      <div style={{ color: "red", fontWeight: "bold" }}>{value}</div>
    </Html>
  );
}

// ---------- PLAYER ----------
function Player({ position }) {
  const ref = useRef();
  const keys = useRef({});

  useEffect(() => {
    const down = (e) => (keys.current[e.key] = true);
    const up = (e) => (keys.current[e.key] = false);
    window.addEventListener("keydown", down);
    window.addEventListener("keyup", up);
    return () => {
      window.removeEventListener("keydown", down);
      window.removeEventListener("keyup", up);
    };
  }, []);

  useFrame(() => {
    if (!ref.current) return;
    if (keys.current["w"]) ref.current.position.z -= 0.12;
    if (keys.current["s"]) ref.current.position.z += 0.12;
    if (keys.current["a"]) ref.current.position.x -= 0.12;
    if (keys.current["d"]) ref.current.position.x += 0.12;
  });

  return (
    <Box ref={ref} position={position} args={[1, 1, 1]}>
      <meshStandardMaterial color="#4f83ff" />
    </Box>
  );
}

// ---------- PET ----------
function Pet({ type }) {
  const colors = { fire: "orange", water: "deepskyblue", grass: "limegreen" };
  return (
    <Sphere args={[0.6, 32, 32]} position={[1.5, 0.3, 0]}>
      <meshStandardMaterial emissive={colors[type]} color={colors[type]} />
    </Sphere>
  );
}

// ---------- ENEMY ----------
function Enemy({ hp }) {
  return (
    <Box position={[-2, 0, 0]} args={[1.2, 1.2, 1.2]}>
      <meshStandardMaterial color={hp > 0 ? "crimson" : "gray"} />
    </Box>
  );
}

// ---------- WORLD ----------
function World() {
  return (
    <>
      <Plane rotation={[-Math.PI / 2, 0, 0]} args={[100, 100]}>
        <meshStandardMaterial color="#7ec850" />
      </Plane>

      {[...Array(25)].map((_, i) => (
        <Box key={i} position={[rand(50) - 25, 0.5, rand(50) - 25]} args={[1, 1, 1]}>
          <meshStandardMaterial color="green" />
        </Box>
      ))}
    </>
  );
}

export default function Game() {
  const [grade, setGrade] = useState(null);
  const [pet, setPet] = useState(null);
  const [mode, setMode] = useState("menu");
  const [question, setQuestion] = useState(null);
  const [input, setInput] = useState("");
  const [playerHP, setPlayerHP] = useState(30);
  const [enemyHP, setEnemyHP] = useState(10);
  const [xp, setXP] = useState(0);
  const [level, setLevel] = useState(1);
  const [damageText, setDamageText] = useState(null);

  const startBattle = () => {
    setEnemyHP(10 + level * 3);
    setQuestion(generateQuestion(grade));
    setMode("battle");
  };

  const submit = () => {
    if (parseInt(input) === question.a) {
      setEnemyHP((h) => h - 5);
      setXP((x) => x + 15);
      setDamageText("-5");
    } else {
      setPlayerHP((h) => h - 4);
      setDamageText("-4");
    }
    setTimeout(() => setDamageText(null), 800);
    setInput("");
    setQuestion(generateQuestion(grade));
  };

  useEffect(() => {
    if (xp >= 40) {
      setXP(0);
      setLevel((l) => l + 1);
    }
  }, [xp]);

  // ---------- UI ----------
  if (!grade) {
    return (
      <div className="p-4">
        <h1>Choose Grade</h1>
        {[1,2,3,4,5].map(g => (
          <button key={g} onClick={() => setGrade(g)} className="m-2 p-2 bg-blue-500 text-white rounded-xl">
            Grade {g}
          </button>
        ))}
      </div>
    );
  }

  if (!pet) {
    return (
      <div className="p-4">
        <h1>Choose Pet</h1>
        {["fire","water","grass"].map(p => (
          <button key={p} onClick={() => setPet(p)} className="m-2 p-2 bg-green-500 text-white rounded-xl">
            {p}
          </button>
        ))}
      </div>
    );
  }

  if (mode === "menu") {
    return (
      <div className="p-4">
        <h1>🌍 Open World</h1>
        <p>Level: {level} | XP: {xp}</p>
        <button onClick={startBattle} className="p-2 bg-red-500 text-white rounded-xl shadow">
          Enter Battle
        </button>

        <Canvas camera={{ position: [0, 12, 12] }}>
          <ambientLight intensity={0.7} />
          <pointLight position={[10, 15, 10]} />
          <World />
          <Player position={[0, 0.5, 0]} />
          <Pet type={pet} />
          <OrbitControls />
        </Canvas>
      </div>
    );
  }

  // ---------- BATTLE ----------
  return (
    <div className="h-screen">
      <div className="absolute top-0 left-0 bg-white p-4 rounded-xl shadow">
        <p>❤️ Player HP: {playerHP}</p>
        <p>👾 Enemy HP: {enemyHP}</p>
        <p>⭐ Level: {level}</p>
        <p>Solve: {question?.q}</p>
        <input value={input} onChange={(e)=>setInput(e.target.value)} className="border p-1 rounded" />
        <button onClick={submit} className="ml-2 p-1 bg-blue-500 text-white rounded">Attack</button>
        <button onClick={()=>setMode("menu")} className="ml-2 p-1 bg-gray-500 text-white rounded">Run</button>
      </div>

      <Canvas camera={{ position: [0, 3, 7] }}>
        <ambientLight />
        <pointLight position={[10, 10, 10]} />
        <Player position={[0, 0, 0]} />
        <Pet type={pet} />
        <Enemy hp={enemyHP} />
        {damageText && <DamageText value={damageText} position={[0,2,0]} />}
        <OrbitControls />
      </Canvas>
    </div>
  );
}
