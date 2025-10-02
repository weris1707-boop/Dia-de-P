<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Día de Picnic</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root{
      --rojo:#ff9999; --rojo-osc:#ff4d4d;
      --azul:#99ccff; --azul-osc:#3399ff;
      --verde:#99ff99; --verde-osc:#33cc33;
      --naranja:#ffcc99; --naranja-osc:#ff9933;
      --miel1:#FFD700; --miel2:#FFA500;
    }
    *{box-sizing:border-box}
    body{
      font-family: system-ui, Arial, sans-serif;
      display:flex; flex-direction:column; align-items:center;
      min-height:100vh; margin:0; padding:16px;
      background:
        linear-gradient(45deg,#ff4d4d 25%,transparent 25%),
        linear-gradient(-45deg,#ff4d4d 25%,transparent 25%),
        linear-gradient(45deg,transparent 75%,#ff4d4d 75%),
        linear-gradient(-45deg,transparent 75%,#ff4d4d 75%);
      background-size:80px 80px; background-color:#fff;
    }
    #tituloJuego{
      font-size:44px; font-weight:900; color:#ff9800; text-align:center;
      margin:8px 0 16px; text-shadow:3px 3px 6px rgba(0,0,0,.25);
      background:rgba(255,255,255,.9); padding:10px 18px; border-radius:16px;
    }
    #configuracion{
      width:min(900px,95vw); background:rgba(255,255,255,.95);
      border-radius:14px; padding:16px; box-shadow:0 0 10px rgba(0,0,0,.15);
    }
    #configuracion .fila{display:flex; gap:12px; align-items:center; flex-wrap:wrap}
    select,input[type="checkbox"]{font-size:16px; padding:6px}
    button{cursor:pointer; border:none; border-radius:10px; font-weight:700}
    #iniciar,#reiniciar{
      background:#007bff; color:#fff; padding:10px 18px; font-size:18px;
    }
    #iniciar:hover,#reiniciar:hover{background:#0056b3}
    #reglas{
      margin-top:12px; background:#fff; border-radius:10px; padding:12px;
      max-height:220px; overflow:auto; border:1px solid #eee
    }
    #barraInfo{display:flex; gap:12px; align-items:center; margin:14px 0; flex-wrap:wrap}
    #marcador,#jugadorTurno,#mensaje{
      background:rgba(255,255,255,.9); padding:8px 12px; border-radius:12px; font-weight:800;
    }
    #contenedorJuego{
      display:flex; gap:16px; align-items:flex-start; justify-content:center; flex-wrap:wrap;
      width:100%; margin-top:8px;
    }

    /* Panal */
    .hex-grid{display:flex; flex-direction:column; gap:6px}
    .hex-row{display:flex; gap:6px}
    .hex-row:nth-child(even){margin-left:40px}
    .hex{
      position:relative; width:70px; height:80px;
      background:linear-gradient(to bottom,var(--miel1),var(--miel2));
      clip-path:polygon(50% 0%,93% 25%,93% 75%,50% 100%,7% 75%,7% 25%);
      display:flex; align-items:center; justify-content:center;
      font-size:20px; font-weight:900; border:2px solid #444;
      transition:transform .15s, background .25s, filter .2s;
      user-select:none; cursor:pointer;
    }
    .hex:hover{transform:translateY(-2px)}
    .hex.seleccionado{
      background:#eee!important; border:2px solid #000; cursor:not-allowed;
    }
    .hex.seleccionado::after{
      content:"🐝"; position:absolute; top:-16px; font-size:22px;
      filter: drop-shadow(0 1px 1px rgba(0,0,0,.25));
    }
    .hex.bloqueada{filter:grayscale(1) brightness(.85)}

    /* Comodines a los lados */
    #comodinesArea{display:flex; gap:16px; align-items:flex-start}
    .comodinesJugador{
      background:rgba(255,255,255,.95); padding:12px; border-radius:12px;
      box-shadow:0 0 8px rgba(0,0,0,.2); width:190px;
      display:flex; flex-direction:column; align-items:center; gap:8px;
      border:3px solid transparent;
    }
    .comodinesJugador h3{margin:0; font-size:18px}
    .card{
      width:100%; height:120px; border-radius:12px; padding:10px;
      display:flex; flex-direction:column; align-items:center; justify-content:space-between;
      color:#222; font-weight:800; box-shadow:2px 2px 6px rgba(0,0,0,.25);
      transition:transform .35s, background .25s, filter .2s;
      cursor:pointer; user-select:none; position:relative;
      backface-visibility:hidden; transform-style:preserve-3d;
      text-align:center;
    }
    .card:hover{transform:scale(1.06)}
    .card-icon{font-size:34px}
    .card-text{font-size:14px; line-height:1.1}
    .card.usado{
      background:#bbb!important; color:#666!important; cursor:not-allowed;
      transform:rotateY(180deg) scale(.94); box-shadow:none; filter:grayscale(30%);
    }
    /* Colores por jugador */
    .jugador1{border-color:var(--rojo)}    .jugador1 .card{background:linear-gradient(135deg,var(--rojo),var(--rojo-osc))}
    .jugador2{border-color:var(--azul)}    .jugador2 .card{background:linear-gradient(135deg,var(--azul),var(--azul-osc))}
    .jugador3{border-color:var(--verde)}   .jugador3 .card{background:linear-gradient(135deg,var(--verde),var(--verde-osc))}
    .jugador4{border-color:var(--naranja)} .jugador4 .card{background:linear-gradient(135deg,var(--naranja),var(--naranja-osc))}

    /* Overlay Victoria */
    #victoriaOverlay{
      position:fixed; inset:0; background:rgba(0,0,0,.65);
      display:none; align-items:center; justify-content:center; z-index:1000;
    }
    #victoriaCard{
      position:relative; width:min(680px,92vw);
      padding:48px 28px 64px; text-align:center;
      border-radius:22px; color:#3b2b00; font-weight:900; font-size:40px;
      background: linear-gradient(to bottom, var(--miel1), var(--miel2));
      box-shadow: 0 14px 40px rgba(255, 152, 0, .65), inset 0 -6px 0 rgba(0,0,0,.1);
      overflow:hidden;
    }
    #victoriaCard::after{
      content:""; position:absolute; left:0; top:0; width:100%; height:120%;
      background:
        radial-gradient(20px 40px at 10% 0%, rgba(255,183,0,.95) 60%, transparent 61%) 0 0/120px 120px,
        radial-gradient(22px 44px at 35% 0%, rgba(255,183,0,.95) 60%, transparent 61%) 0 0/140px 140px,
        radial-gradient(18px 36px at 60% 0%, rgba(255,183,0,.95) 60%, transparent 61%) 0 0/110px 110px,
        radial-gradient(24px 48px at 85% 0%, rgba(255,183,0,.95) 60%, transparent 61%) 0 0/150px 150px;
      animation: drip 3.6s linear infinite; opacity:.9;
    }
    @keyframes drip{
      0%{ transform: translateY(-5%); }
      50%{ transform: translateY(3%); }
      100%{ transform: translateY(-5%); }
    }
    #victoriaTexto{position:relative; z-index:2; text-shadow: 0 2px 0 rgba(255,255,255,.7), 0 8px 24px rgba(0,0,0,.25);}

    /* Volumen */
    #sonidoControles{
      display:flex; align-items:center; gap:8px;
      background:rgba(255,255,255,.9); padding:6px 10px; border-radius:10px;
    }
    #volumen{width:100px}
  </style>
</head>
<body>
  <div id="tituloJuego">🌻 Día de Picnic 🐝</div>

  <div id="configuracion">
    <div class="fila">
      <label for="numJugadores">Número de jugadores:</label>
      <select id="numJugadores">
        <option value="2">2</option>
        <option value="3">3</option>
        <option value="4">4</option>
      </select>
      <label><input type="checkbox" id="jugarBot"> Jugar contra Bot 🤖</label>
      <button id="iniciar">▶️ Iniciar Juego</button>
    </div>
    <div id="reglas">
      <h3>📜 Reglas</h3>
      <ul>
        <li>El objetivo es llegar exactamente a <b>58</b>.</li>
        <li>La <b>suma es compartida</b> por todos los jugadores.</li>
        <li>Para ganar debes llegar a 58 habiendo usado <b>tus 3 comodines</b>.</li>
        <li>Cada jugador recibe <b>3 comodines aleatorios</b> al inicio.</li>
      </ul>
    </div>
  </div>

  <div id="barraInfo" style="display:none;">
    <div id="marcador">Suma: 0</div>
    <div id="jugadorTurno"></div>
    <button id="reiniciar">🔄 Reiniciar</button>
    <div id="sonidoControles">
      🔊 <input type="range" id="volumen" min="0" max="1" step="0.05" value="1">
      <button id="mute">🔇</button>
    </div>
  </div>

  <div id="contenedorJuego" style="display:none;">
    <div id="comodinesArea"></div>
    <div id="tabla" class="hex-grid"></div>
  </div>

  <div id="mensaje"></div>

  <!-- 🎵 Sonidos -->
  <audio id="sndSeleccion" preload="auto" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
  <audio id="sndComodin"   preload="auto" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
  <audio id="sndVictoria"  preload="auto" src="https://actions.google.com/sounds/v1/ambiences/cheer.ogg"></audio>

  <!-- 🎉 Overlay Victoria -->
  <div id="victoriaOverlay">
    <div id="victoriaCard"><div id="victoriaTexto"></div></div>
  </div>

  <script>
    const filas=8, columnas=7, objetivo=58;
    const colores=["#ff9999","#99ccff","#99ff99","#ffcc99"];
    const tabla=document.getElementById("tabla");
    const marcador=document.getElementById("marcador");
    const jugadorTurno=document.getElementById("jugadorTurno");
    const reiniciarBtn=document.getElementById("reiniciar");
    const iniciarBtn=document.getElementById("iniciar");
    const numJugadoresSelect=document.getElementById("numJugadores");
    const configuracion=document.getElementById("configuracion");
    const contenedorJuego=document.getElementById("contenedorJuego");
    const barraInfo=document.getElementById("barraInfo");
    const comodinesArea=document.getElementById("comodinesArea");
    const jugarBotCheckbox=document.getElementById("jugarBot");
    const mensaje=document.getElementById("mensaje");

    // Sonidos
    const SND={seleccion:document.getElementById("sndSeleccion"),comodin:document.getElementById("sndComodin"),victoria:document.getElementById("sndVictoria")};
    let muted=false;
    function playSound(k){if(!muted){SND[k].currentTime=0;SND[k].play();}}
    document.getElementById("volumen").addEventListener("input",e=>{
      const v=e.target.value;
      Object.values(SND).forEach(s=>s.volume=v);
    });
    document.getElementById("mute").addEventListener("click",()=>{
      muted=!muted;
      document.getElementById("mute").textContent=muted?"🔊":"🔇";
    });

    let numJugadores=2,jugarBot=false,botIndex=null,turno=0,suma=0,gameOver=false;
    let estadoJugadores=[];

    const listaComodines=[
      {id:"multiplicar",icon:"✖️",texto:"Multiplicar una vez"},
      {id:"restar",icon:"➖",texto:"Restar una vez"},
      {id:"dobleTurno",icon:"🔁",texto:"Turno doble"}
    ];

    iniciarBtn.onclick=iniciarJuego;
    reiniciarBtn.onclick=()=>location.reload();

    function iniciarJuego(){
      numJugadores=parseInt(numJugadoresSelect.value,10);
      jugarBot=jugarBotCheckbox.checked;
      if(jugarBot)botIndex=numJugadores-1;
      suma=0;turno=0;gameOver=false;mensaje.textContent="";
      crearTablero();crearComodines();
      marcador.textContent="Suma: 0";
      jugadorTurno.textContent="Turno del Jugador 1";
      jugadorTurno.style.color=colores[0];
      configuracion.style.display="none";
      contenedorJuego.style.display="flex";
      barraInfo.style.display="flex";
    }

    function crearTablero(){
      tabla.innerHTML="";
      for(let i=0;i<filas;i++){
        const row=document.createElement("div");
        row.className="hex-row";
        for(let j=0;j<columnas;j++){
          const c=document.createElement("div");
          c.className="hex";
          c.textContent=(j%7)+1;
          c.onclick=()=>onCelda(c);
          row.appendChild(c);
        }
        tabla.appendChild(row);
      }
    }

    function crearComodines(){
      comodinesArea.innerHTML="";estadoJugadores=[];
      for(let p=0;p<numJugadores;p++){
        const panel=document.createElement("div");
        panel.className=`comodinesJugador jugador${p+1}`;
        panel.style.borderColor=colores[p];
        panel.innerHTML=`<h3>🎴 J${p+1}${(jugarBot&&p===botIndex)?" 🤖":""}</h3>`;
        const pool=[...listaComodines];
        const cartas=[];
        for(let k=0;k<3;k++){const idx=Math.floor(Math.random()*pool.length);cartas.push(pool[idx]);pool.splice(idx,1);}
        const usados={};cartas.forEach(c=>usados[c.id]=false);
        cartas.forEach(c=>{
          const card=document.createElement("div");
          card.className="card";card.dataset.id=c.id;card.dataset.jugador=p;
          card.innerHTML=`<div class="card-icon">${c.icon}</div><div class="card-text">${c.texto}</div>`;
          card.onclick=()=>usarComodin(p,c.id,card);
          panel.appendChild(card);
        });
        comodinesArea.appendChild(panel);
        estadoJugadores.push({comodines:usados,multiplicar:false,restar:false});
      }
    }

    function usarComodin(j,id,card){
      if(gameOver||j!==turno)return;
      const est=estadoJugadores[j];
      if(est.comodines[id])return;
      est.comodines[id]=true;
      card.classList.add("usado");
      playSound("comodin");
      if(id==="multiplicar")est.multiplicar=true;
      if(id==="restar")est.restar=true;
      if(id==="dobleTurno")est.dobleTurno=true;
    }

    function onCelda(c){
      if(gameOver||c.classList.contains("seleccionado"))return;
      let v=parseInt(c.textContent,10);
      const est=estadoJugadores[turno];
      if(est.multiplicar){v*=2;est.multiplicar=false;}
      if(est.restar){v*=-1;est.restar=false;}
      suma+=v;marcador.textContent="Suma: "+suma;
      c.classList.add("seleccionado");c.style.background=colores[turno];
      playSound("seleccion");
      if(suma===objetivo&&usadosTodos(turno)){victoria(turno);return;}
      avanzar();
    }

    function usadosTodos(j){return Object.values(estadoJugadores[j].comodines).every(x=>x);}
    function avanzar(){
      if(gameOver)return;
      const est=estadoJugadores[turno];
      if(est.dobleTurno){est.dobleTurno=false;return;}
      turno=(turno+1)%numJugadores;
      jugadorTurno.textContent="Turno del Jugador "+(turno+1);
      jugadorTurno.style.color=colores[turno];
      if(jugarBot&&turno===botIndex)setTimeout(turnoBot,900);
    }

    function turnoBot(){
      const est=estadoJugadores[botIndex];
      const pool=Object.keys(est.comodines).filter(k=>!est.comodines[k]);
      if(pool.length&&Math.random()<0.4){
        const id=pool[Math.floor(Math.random()*pool.length)];
        const carta=document.querySelector(`.card[data-jugador="${botIndex}"][data-id="${id}"]`);
        usarComodin(botIndex,id,carta);
      }
      const libres=[...document.querySelectorAll(".hex:not(.seleccionado)")];
      if(libres.length===0)return;
      const c=libres[Math.floor(Math.random()*libres.length)];
      onCelda(c);
    }

    function victoria(j){
      gameOver=true;playSound("victoria");
      const overlay=document.getElementById("victoriaOverlay");
      const texto=document.getElementById("victoriaTexto");
      texto.innerHTML=`🎉 ¡Jugador ${j+1}${(jugarBot&&j===botIndex)?" 🤖":""} gana el picnic! 🍯`;
      overlay.style.display="flex";
    }
  </script>
</body>
</html>
