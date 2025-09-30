/**
 * quantum_simulator.ts
 *
 * Tiny quantum circuit simulator using complex vectors.
 * Supports single-qubit gates and measurement probabilities.
 *
 * Run:
 *   ts-node src/quantum_simulator.ts
 */
type Complex = { r:number; i:number };
function add(a:Complex,b:Complex){return {r:a.r+b.r,i:a.i+b.i};}
function mul(a:Complex,b:Complex){return {r:a.r*b.r - a.i*b.i, i: a.r*b.i + a.i*b.r};}
function conj(a:Complex){return {r:a.r, i:-a.i};}
function abs2(a:Complex){return a.r*a.r + a.i*a.i;}

function H(): Complex[][] { const s = 1/Math.sqrt(2); return [[{r:s,i:0},{r:s,i:0}], [{r:s,i:0},{r:-s,i:0}]]; }
function X(): Complex[][] { return [[{r:0,i:0},{r:1,i:0}], [{r:1,i:0},{r:0,i:0}]]; }

function applyGate(state:Complex[], matrix:Complex[][], qubitIdx:number, nQubits:number){
  const out = state.slice().map(_=>({r:0,i:0}));
  for (let i=0;i<state.length;i++){
    const bit = (i >> (nQubits-qubitIdx-1)) & 1;
    for (let k=0;k<2;k++){
      const target = (bit === k) ? i : (i ^ (1 << (nQubits-qubitIdx-1)));
      const m = matrix[k][bit];
      out[target] = add(out[target], mul(m, state[i]));
    }
  }
  return out;
}

function initState(n:number){
  const len = 1<<n;
  const s: Complex[] = Array(len).fill({r:0,i:0}).map(()=>({r:0,i:0}));
  s[0] = {r:1,i:0};
  return s;
}

function measureProbs(state:Complex[]){ return state.map(s => abs2(s)); }

(function demo(){
  const n=2;
  let state = initState(n);
  // Apply H on qubit 0, then CNOT via sequence (simulate with X on q1 conditioned - omitted for brevity)
  state = applyGate(state, H(), 0, n);
  console.log('Probabilities after H on q0:', measureProbs(state).map(p=>p.toFixed(3)));
})();
