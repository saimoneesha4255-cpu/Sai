<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Housey (90-ball) — Mini Game</title>
<style>
  :root{
    --bg:#f4f7fb; --card:#ffffff; --accent:#2563eb; --muted:#6b7280;
    --good:#059669; --danger:#ef4444;
  }
  body{font-family:Inter,ui-sans-serif,system-ui,Segoe UI,Roboto,Helvetica,Arial; background:var(--bg); margin:0; padding:24px; color:#111827;}
  .wrap{max-width:1100px;margin:0 auto;display:grid;grid-template-columns:360px 1fr;gap:18px;}
  .panel{background:var(--card);padding:16px;border-radius:12px;box-shadow:0 6px 18px rgba(15,23,42,0.06);}
  h2{margin:0 0 12px 0;font-size:18px;}
  label{display:block;font-size:13px;color:var(--muted);margin-bottom:6px;}
  input[type="text"]{width:100%;padding:8px;border-radius:8px;border:1px solid #e6e9ef;}
  button{background:var(--accent);color:white;border:none;padding:8px 12px;border-radius:8px;cursor:pointer;}
  button.ghost{background:transparent;color:var(--accent);border:1px solid #e6eeff;}
  .players-list{margin-top:12px;}
  .player-item{display:flex;align-items:center;justify-content:space-between;padding:8px;border-radius:8px;border:1px dashed #eef2ff;margin-bottom:8px;}
  .player-meta{font-size:13px;}
  .ticket{display:inline-block;background:#fff;padding:8px;border-radius:8px;margin:6px;box-shadow:0 3px 8px rgba(2,6,23,0.06);}
  .ticket-grid{display:grid;grid-template-columns:repeat(9,32px);gap:4px;padding:6px;background:linear-gradient(180deg,#fbfdff,#fff);border-radius:6px;}
  .cell{width:32px;height:28px;display:flex;align-items:center;justify-content:center;border-radius:4px;font-size:12px;background:#f8fafc;color:#111;}
  .cell.blank{background:transparent;}
  .cell.marked{text-decoration:line-through;color:var(--muted);opacity:0.5;}
  .draw-area{display:flex;gap:10px;align-items:center;flex-wrap:wrap;}
  .big-number{width:70px;height:70px;border-radius:10px;display:flex;align-items:center;justify-content:center;font-size:20px;background:linear-gradient(180deg,#fff,#f3f7ff);box-shadow:0 8px 22px rgba(37,99,235,0.12);}
  .drawn-list{display:flex;flex-wrap:wrap;gap:6px;margin-top:8px;}
  .chip{padding:6px 8px;border-radius:999px;border:1px solid #e6eefb;background:white;font-size:12px;}
  .chip.drawn{background:#eef6ff;border-color:#dbeefe;}
  .controls{display:flex;gap:8px;margin-top:8px;flex-wrap:wrap;}
  .hint{font-size:13px;color:var(--muted);margin-top:8px;}
  .player-top{display:flex;gap:8px;align-items:center;}
  .code{padding:6px 8px;border-radius:8px;background:#f1f5f9;font-size:12px;border:1px solid #e6eefb;}
  .small{font-size:12px;color:var(--muted)}
  .footer{margin-top:12px;font-size:13px;color:var(--muted)}
</style>
</head>
<body>
<div class="wrap">
  <div class="panel">
    <h2>Create / Join Player</h2>
    <div>
      <label>Player name</label>
      <input id="playerName" type="text" placeholder="e.g., Alice" />
    </div>
    <div style="margin-top:8px;">
      <button id="createPlayerBtn">Create Player (make code + ticket)</button>
      <button id="joinByCodeBtn" class="ghost" style="margin-left:8px">Join by code</button>
    </div>

    <div id="joinArea" style="display:none;margin-top:8px;">
      <label>Enter code to join</label>
      <input id="joinCodeInput" type="text" placeholder="player-code-xxxxx" />
      <div style="margin-top:8px;">
        <button id="joinConfirmBtn">Join</button>
      </div>
    </div>

    <div class="players-list" id="playersList" style="margin-top:12px;">
      <h3 style="margin:6px 0;font-size:14px">Players</h3>
      <!-- player items appended -->
    </div>

    <div style="margin-top:12px;">
      <h3 style="font-size:14px;margin-bottom:6px">Game Controls</h3>
      <div class="controls">
        <button id="drawNext">Draw Next</button>
        <button id="autoDraw" class="ghost">Auto-Draw</button>
        <button id="stopAuto" class="ghost" style="display:none">Stop Auto</button>
        <button id="resetGame" class="ghost">Reset Game</button>
      </div>
      <div class="hint">Drawn numbers are unique and taken from 1–90. Tickets auto-mark drawn numbers. You can also click a number to toggle mark.</div>
    </div>

    <div style="margin-top:12px;">
      <h3 style="font-size:14px;margin-bottom:6px">Draw</h3>
      <div class="draw-area">
        <div class="big-number" id="lastDraw">—</div>
        <div>
          <div class="small">Remaining</div>
          <div id="remainingCount" style="font-weight:600">90</div>
        </div>
      </div>
      <div class="drawn-list" id="drawnList"></div>
    </div>

    <div class="footer">Tip: create 2+ players to see different tickets. Tickets follow 90-ball (3×9) classic layout.</div>
  </div>

  <div class="panel">
    <div style="display:flex;justify-content:space-between;align-items:center;">
      <h2>Tickets</h2>
      <div class="player-top">
        <div class="small">Selected player:</div>
        <div id="selectedPlayerName" class="code" style="margin-left:8px">None</div>
      </div>
    </div>

    <div id="ticketsArea" style="margin-top:12px;display:flex;flex-wrap:wrap;">
      <!-- tickets will render here -->
    </div>
  </div>
</div>

<script>
/*
 Housey (90-ball) simple web implementation
 - Tickets: 3 rows x 9 columns, 15 numbers (5 per row)
 - Draw numbers from 1..90 without repetition
 - Auto marking on match
*/

// ---------- Utility functions ----------
function uid(len=6){
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  let s=''; for(let i=0;i<len;i++) s+=chars[Math.floor(Math.random()*chars.length)];
  return s;
}
function shuffle(arr){ for(let i=arr.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [arr[i],arr[j]]=[arr[j],arr[i]];} return arr; }

// ---------- Ticket generation (UK-style 90-ball) ----------
function generate90ballTicket(){
  // We'll use a common approach: 3 rows x 9 columns, each column owns a specific number range:
  // col 0 -> 1-9, col 1 -> 10-19, ..., col 8 -> 80-90
  // Each row contains exactly 5 numbers -> total 15 numbers; each column can have 1-3 numbers, overall 15 spots.
  const columns = [];
  for(let c=0;c<9;c++){
    const start = c*10 + 1;
    const end = (c===0) ? 9 : (c===8 ? 90 : c*10+9); // first column 1-9, last 80-90
    const range = [];
    for(let n=start;n<=end;n++) range.push(n);
    columns.push(shuffle(range));
  }
  // We need to pick numbers for each column such that total numbers = 15 and each row has 5 numbers.
  // Approach (simple workable heuristic):
  // 1) Decide how many numbers per column: initially 0.
  const perCol = Array(9).fill(0);
  // Guarantee at least 1 number in 6 columns and spread others so sum = 15.
  // More straightforward: iterate 15 times and pick a column that hasn't exceeded 3 numbers and won't make it impossible to fill rows.
  for(let i=0;i<15;i++){
    let tries=0;
    while(true){
      const c = Math.floor(Math.random()*9);
      if(perCol[c]>=3){ tries++; if(tries>200) break; else continue; }
      perCol[c]++;
      break;
    }
  }
  // If any column zero — it's fine. Now we need to place numbers into 3x9 grid ensuring each row has 5 numbers.
  // We'll create an array of column picks: for each column, pick perCol[c] numbers from columns[c].
  const colNumbers = perCol.map((count,c)=> shuffle(columns[c]).slice(0,count));
  // Now assign those numbers into 3 rows trying to keep 5 per row.
  const rows = [Array(9).fill(null), Array(9).fill(null), Array(9).fill(null)];
  // For each column, distribute its numbers into rows: randomly pick rows that are not yet full (<=5).
  const counts = [0,0,0];
  for(let c=0;c<9;c++){
    const nums = colNumbers[c];
    // shuffle rows available positions
    const rowOrder = shuffle([0,1,2]);
    for(let i=0;i<nums.length;i++){
      // choose a row among rowOrder with counts < 5 and cell empty
      let placed=false;
      for(const r of rowOrder){
        if(counts[r]<5 && rows[r][c]===null){
          rows[r][c]=nums[i];
          counts[r]++; placed=true; break;
        }
      }
      if(!placed){
        // fallback: place in any row with empty cell
        for(let r=0;r<3;r++){
          if(rows[r][c]===null){ rows[r][c]=nums[i]; counts[r]++; break; }
        }
      }
    }
  }
  // after distribution, some rows might not have 5 numbers. Fix by moving numbers if necessary:
  // If a row has <5, find columns where other rows have >5 and swap.
  for(let attempt=0; attempt<200 && (counts[0]!==5 || counts[1]!==5 || counts[2]!==5); attempt++){
    for(let r=0;r<3;r++){
      while(counts[r]>5){
        // move one of its numbers to a row that has <5 in a different column
        let moved=false;
        for(let c=0;c<9 && counts[r]>5;c++){
          if(rows[r][c]!==null){
            for(let r2=0;r2<3;r2++){
              if(r2!==r && counts[r2]<5 && rows[r2][c]===null){
                rows[r2][c]=rows[r][c];
                rows[r][c]=null;
                counts[r]--; counts[r2]++; moved=true; break;
              }
            }
          }
        }
        if(!moved) break;
      }
      while(counts[r]<5){
        // find a column where some other row has >5 or any non-null in same col that can move
        let moved=false;
        for(let c=0;c<9 && counts[r]<5;c++){
          for(let r2=0;r2<3;r2++){
            if(r2!==r && counts[r2]>5 && rows[r2][c]!==null && rows[r][c]===null){
              rows[r][c]=rows[r2][c];
              rows[r2][c]=null;
              counts[r]++; counts[r2]--; moved=true; break;
            }
          }
        }
        if(!moved) break;
      }
    }
  }

  // Final sanitized ticket: each row array with number or null
  // Sort numbers in each column top-to-bottom? Typical ticket numbers in a column are sorted top->down; we'll sort numbers within each column to match that
  // Build column arrays and reorder rows so numbers increase top->bottom where applicable
  for(let c=0;c<9;c++){
    const pairs=[];
    for(let r=0;r<3;r++) if(rows[r][c]!==null) pairs.push(rows[r][c]);
    pairs.sort((a,b)=>a-b);
    // fill them back top-down
    let idx=0;
    for(let r=0;r<3;r++){
      if(rows[r][c]!==null){
        rows[r][c]=pairs[idx++]; 
      }
    }
  }

  // Flatten to object structure
  const cells = [];
  for(let r=0;r<3;r++){
    for(let c=0;c<9;c++){
      cells.push(rows[r][c]===null ? null : rows[r][c]);
    }
  }
  return {cells}; // 27 cells (3*9), null where blank
}

// ---------- Game state ----------
const state = {
  players: {}, // code -> {name, code, ticket:{cells:[]}, marks:Set}
  drawn: [],   // numbers drawn in order
  remaining: Array.from({length:90},(_,i)=>i+1),
  autoInterval: null
};

// ---------- DOM references ----------
const playersListEl = document.getElementById('playersList');
const ticketsArea = document.getElementById('ticketsArea');
const lastDrawEl = document.getElementById('lastDraw');
const drawnListEl = document.getElementById('drawnList');
const remainingCountEl = document.getElementById('remainingCount');
const selectedPlayerNameEl = document.getElementById('selectedPlayerName');

let selectedPlayerCode = null;

// ---------- Functions to manage state ----------
function updateUI(){
  // Players list
  playersListEl.querySelectorAll('.player-item').forEach(n=>n.remove());
  ticketsArea.innerHTML = '';
  for(const code in state.players){
    const p = state.players[code];
    // player card in left list
    const item = document.createElement('div'); item.className='player-item';
    const meta = document.createElement('div'); meta.className='player-meta';
    meta.innerHTML = `<strong>${escapeHtml(p.name)}</strong><div class="small">code: <span class="code">${code}</span></div>`;
    const actions = document.createElement('div');
    const btnSelect = document.createElement('button'); btnSelect.textContent='Select';
    btnSelect.onclick=()=>{ selectedPlayerCode = code; selectedPlayerNameEl.textContent = p.name + ' ('+code+')'; updateUI(); };
    const btnExport = document.createElement('button'); btnExport.textContent='Export';
    btnExport.className='ghost'; btnExport.onclick=()=> exportTicket(code);
    const btnRegenerate = document.createElement('button'); btnRegenerate.textContent='Regenerate';
    btnRegenerate.className='ghost'; btnRegenerate.onclick=()=> { p.ticket = makeUniqueTicket(); p.marks = new Set(); updateUI(); };
    actions.appendChild(btnSelect); actions.appendChild(btnExport); actions.appendChild(btnRegenerate);
    item.appendChild(meta); item.appendChild(actions);
    playersListEl.appendChild(item);

    // Ticket display on right
    const ticketWrap = document.createElement('div'); ticketWrap.className='ticket';
    const title = document.createElement('div'); title.style.fontSize='13px'; title.style.marginBottom='6px';
    title.textContent = p.name + ' — ' + code;
    ticketWrap.appendChild(title);
    const grid = document.createElement('div'); grid.className='ticket-grid';
    p.ticket.cells.forEach((val, idx) => {
      const cell = document.createElement('div');
      cell.className = 'cell' + (val===null ? ' blank':'' ) + (p.marks && p.marks.has(val) ? ' marked':'' );
      cell.textContent = val===null ? '' : val;
      // clicking toggles mark (only for numbers)
      if(val!==null){
        cell.style.cursor='pointer';
        cell.onclick = ()=> {
          if(p.marks.has(val)) p.marks.delete(val); else p.marks.add(val);
          updateUI();
        };
      }
      // auto-highlighting drawn numbers visually
      if(state.drawn.includes(val)) {
        // ensure marked
        p.marks.add(val);
      }
      grid.appendChild(cell);
    });
    ticketWrap.appendChild(grid);
    ticketsArea.appendChild(ticketWrap);

    // highlight border if selected
    if(selectedPlayerCode===code){
      ticketWrap.style.boxShadow = '0 12px 30px rgba(37,99,235,0.12)';
    }
  }

  // Drawn list
  drawnListEl.innerHTML='';
  state.drawn.forEach(n=>{
    const chip = document.createElement('div'); chip.className='chip drawn'; chip.textContent = n;
    drawnListEl.appendChild(chip);
  });
  lastDrawEl.textContent = state.drawn.length ? state.drawn[state.drawn.length-1] : '—';
  remainingCountEl.textContent = state.remaining.length;
}

function makeUniqueTicket(){
  // produce a ticket not already present for an existing player
  const existingSerialized = new Set(Object.values(state.players).map(p => JSON.stringify(p.ticket.cells)));
  let attempt=0;
  while(true){
    const t = generate90ballTicket();
    const s = JSON.stringify(t.cells);
    if(!existingSerialized.has(s)) return t;
    attempt++;
    if(attempt>200) { // fallback return anyway
      return t;
    }
  }
}

function createPlayer(name){
  const code = 'PC-'+uid(5);
  const ticket = makeUniqueTicket();
  state.players[code] = {name: name||('Player-'+Object.keys(state.players).length+1), code, ticket, marks: new Set()};
  selectedPlayerCode = code;
  return code;
}

function joinPlayerByCode(code, nameIfNew){
  if(state.players[code]) {
    selectedPlayerCode = code;
    return code;
  } else {
    // let them join with the provided code (rare) — create new player with that code (but ensure code not clashing)
    const ticket = makeUniqueTicket();
    state.players[code] = {name: nameIfNew||('Player-'+Object.keys(state.players).length+1), code, ticket, marks: new Set()};
    selectedPlayerCode = code;
    return code;
  }
}

// Draw next number
function drawNextNumber(){
  if(state.remaining.length===0) { alert('All numbers drawn'); return; }
  const idx = Math.floor(Math.random()*state.remaining.length);
  const num = state.remaining.splice(idx,1)[0];
  state.drawn.push(num);
  // auto-mark on all tickets
  for(const code in state.players){
    const p = state.players[code];
    // if p has that number in ticket -> mark
    if(p.ticket.cells.includes(num)) p.marks.add(num);
  }
  updateUI();
  checkQuickWins(num);
}

// Simple win checks (line / house)
// For brevity: check if any player has all numbers marked (house)
function checkQuickWins(lastNum){
  for(const code in state.players){
    const p = state.players[code];
    const ticketNumbers = p.ticket.cells.filter(x=>x!==null);
    const allMarked = ticketNumbers.every(n=>p.marks.has(n));
    if(allMarked){
      setTimeout(()=> alert(`🏆 House! Player ${p.name} (${code}) completed full ticket!`), 80);
    }
    // optional: check any completed row
    for(let r=0;r<3;r++){
      const rowNumbers = p.ticket.cells.slice(r*9, r*9+9).filter(x=>x!==null);
      if(rowNumbers.length>0 && rowNumbers.every(n=>p.marks.has(n))){
        // announce line for that row
        setTimeout(()=> console.log(`Line complete by ${p.name} (row ${r+1})`), 80);
      }
    }
  }
}

function resetGame(){
  if(state.autoInterval) { clearInterval(state.autoInterval); state.autoInterval=null; document.getElementById('autoDraw').style.display=''; document.getElementById('stopAuto').style.display='none';}
  state.drawn = [];
  state.remaining = Array.from({length:90},(_,i)=>i+1);
  for(const code in state.players) state.players[code].marks = new Set();
  updateUI();
}

function exportTicket(code){
  const p = state.players[code];
  if(!p) return;
  let text = `Ticket for ${p.name} (${code})\\n\\n`;
  for(let r=0;r<3;r++){
    let row=[];
    for(let c=0;c<9;c++){
      const val = p.ticket.cells[r*9+c];
      row.push(val===null ? '__' : String(val).padStart(2,' '));
    }
    text += row.join(' ') + '\\n';
  }
  // open in new window/tab for easy copy
  const w = window.open('', '_blank');
  w.document.write('<pre>'+escapeHtml(text)+'</pre>');
  w.document.title = `Ticket - ${p.name}`;
}

// ---------- Helpers ----------
function escapeHtml(s){ return String(s).replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;'); }

// ---------- Attach DOM handlers ----------
document.getElementById('createPlayerBtn').onclick = ()=>{
  const name = document.getElementById('playerName').value.trim() || null;
  createPlayer(name);
  updateUI();
};
document.getElementById('joinByCodeBtn').onclick = ()=>{
  const ja = document.getElementById('joinArea');
  ja.style.display = ja.style.display==='none' ? 'block' : 'none';
};
document.getElementById('joinConfirmBtn').onclick = ()=>{
  const input = document.getElementById('joinCodeInput').value.trim();
  if(!input) { alert('Enter code'); return; }
  joinPlayerByCode(input, document.getElementById('playerName').value.trim()||'');
  updateUI();
};
document.getElementById('drawNext').onclick = drawNextNumber;
document.getElementById('resetGame').onclick = ()=>{ if(confirm('Reset game and clear all drawn numbers?')) resetGame(); };
document.getElementById('autoDraw').onclick = ()=>{
  if(state.autoInterval) return;
  state.autoInterval = setInterval(()=> {
    if(state.remaining.length===0){ clearInterval(state.autoInterval); state.autoInterval=null; document.getElementById('autoDraw').style.display=''; document.getElementById('stopAuto').style.display='none'; return; }
    drawNextNumber();
  }, 1500);
  document.getElementById('autoDraw').style.display='none';
  document.getElementById('stopAuto').style.display='';
};
document.getElementById('stopAuto').onclick = ()=>{
  if(state.autoInterval) { clearInterval(state.autoInterval); state.autoInterval=null; }
  document.getElementById('autoDraw').style.display='';
  document.getElementById('stopAuto').style.display='none';
};

// Initialize with one sample player
createPlayer('Host');
updateUI();

</script>
</body>
</html>

