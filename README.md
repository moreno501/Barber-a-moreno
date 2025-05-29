<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>💈 Reservas Barbería Moreno</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    :root {
      --color-principal: #0d47a1;
      --color-secundario: #b71c1c;
      --color-boton: #08a1da;
      --color-hover: #067dab;
      --color-texto: #fff;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, var(--color-principal), var(--color-secundario));
      color: var(--color-texto);
      padding: 20px;
    }

    h1, h2 {
      text-align: center;
      margin-bottom: 10px;
    }

    .seccion {
      background-color: rgba(255, 255, 255, 0.08);
      border: 1px solid rgba(255, 255, 255, 0.2);
      padding: 15px;
      margin: 20px auto;
      border-radius: 15px;
      max-width: 500px;
      box-shadow: 0 8px 16px rgba(0,0,0,0.2);
      transition: transform 0.3s ease;
    }

    .seccion:hover {
      transform: scale(1.01);
    }

    input, button {
      width: 100%;
      padding: 10px;
      margin: 8px 0;
      border-radius: 10px;
      font-size: 15px;
    }

    input {
      background-color: rgba(255, 255, 255, 0.15);
      border: none;
      color: white;
    }

    input::placeholder {
      color: #ddd;
    }

    button {
      background-color: var(--color-boton);
      border: none;
      color: white;
      font-weight: bold;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }

    button:hover {
      background-color: var(--color-hover);
    }

    ul {
      padding-left: 20px;
    }

    li {
      margin-bottom: 5px;
    }

    .dueño {
      background-color: rgba(255, 255, 255, 0.1);
    }

    .resumen {
      background-color: rgba(0, 0, 0, 0.3);
      padding: 10px;
      border-radius: 10px;
      margin-top: 10px;
    }

    #vistaDueño {
      display: none;
    }

    .icono {
      margin-right: 6px;
    }

    p {
      margin: 4px 0;
    }
  </style>
</head>
<body>
  <h1>💈 Reservas - Barbería Moreno</h1>

  <div class="seccion">
    <h2>1️⃣ Reservar cita</h2>
    <input type="text" id="clienteNombre" placeholder="Nombre y Apellido">
    <button onclick="reservar()">📅 Reservar</button>
    <p id="siguienteId"></p>
  </div>

  <div class="seccion">
    <h2>2️⃣ Cancelar cita</h2>
    <input type="text" id="cancelarId" placeholder="Tu ID">
    <button onclick="cancelarReserva()">❌ Cancelar</button>
    <p id="mensajeCancelacion"></p>
  </div>

  <div class="seccion">
    <h2>3️⃣ Ver horarios disponibles</h2>
    <button onclick="verDisponibles()">🕓 Mostrar horarios</button>
    <ul id="horariosDisponibles"></ul>
  </div>

  <div class="seccion">
    <h2>4️⃣ Consultar tu reserva</h2>
    <input type="text" id="buscarCliente" placeholder="Tu ID">
    <button onclick="buscarComoCliente()">🔍 Buscar</button>
    <p id="resultadoBusquedaCliente"></p>
  </div>

  <div class="seccion">
    <h2>🔐 Iniciar sesión como dueño</h2>
    <input type="password" id="claveDueño" placeholder="Contraseña de dueño">
    <button onclick="iniciarSesionDueño()">🔓 Entrar</button>
    <p id="mensajeDueño"></p>
  </div>

  <div class="seccion dueño" id="vistaDueño">
    <h2>🔒 Vista del dueño</h2>
    <button onclick="verTodasLasReservas()">📖 Ver todas las reservas</button>
    <ul id="reservasDueño"></ul>

    <div class="resumen" id="resumenDueño"></div>
  </div>

  <script>
    const reservas = {};
    const horarios = generarHorarios("09:00", "21:00", 30);
    const usados = new Set();
    let cancelacionesTotales = 0;
    const CLAVE_DUEÑO = "barber123";

    function generarHorarios(inicio, fin, intervaloMin) {
      const result = [];
      let [h, m] = inicio.split(":").map(Number);
      const [hFin, mFin] = fin.split(":").map(Number);
      let totalMin = h * 60 + m;
      const totalMinFin = hFin * 60 + mFin;

      while (totalMin < totalMinFin) {
        const hora = String(Math.floor(totalMin / 60)).padStart(2, "0");
        const min = String(totalMin % 60).padStart(2, "0");
        result.push(`${hora}:${min}`);
        totalMin += intervaloMin;
      }
      return result;
    }

    function obtenerSiguienteIdDisponible() {
      for (let i = 1; i <= 30; i++) {
        if (!usados.has(i.toString())) return i.toString();
      }
      return null;
    }

    function mostrarSiguienteId() {
      const siguiente = obtenerSiguienteIdDisponible();
      document.getElementById("siguienteId").textContent = siguiente
        ? `✅ Siguiente ID disponible: ${siguiente}`
        : "❌ No hay más IDs disponibles.";
    }

    function reservar() {
      const nombre = document.getElementById("clienteNombre").value.trim();
      const siguienteId = obtenerSiguienteIdDisponible();
      if (!nombre) return alert("⚠️ Ingresa tu nombre.");
      if (!siguienteId) return alert("⚠️ No hay más IDs disponibles.");

      const horariosReservados = Object.values(reservas).map(r => r.horario);
      const siguienteHorario = horarios.find(h => !horariosReservados.includes(h));
      if (!siguienteHorario) return alert("⚠️ No hay horarios disponibles.");

      let nombreFinal = nombre;
      let contador = 1;
      while (Object.values(reservas).some(r => r.nombre === nombreFinal)) {
        nombreFinal = nombre + (++contador);
      }

      reservas[siguienteId] = { nombre: nombreFinal, horario: siguienteHorario };
      usados.add(siguienteId);

      alert(`✅ Reserva hecha para ${nombreFinal} a las ${siguienteHorario}. Tu ID es ${siguienteId}.`);
      mostrarSiguienteId();
    }

    function cancelarReserva() {
      const id = document.getElementById("cancelarId").value.trim();
      const mensaje = document.getElementById("mensajeCancelacion");
      if (reservas[id]) {
        delete reservas[id];
        usados.delete(id);
        cancelacionesTotales++;
        mensaje.textContent = `✅ Reserva cancelada para ID ${id}.`;
        mostrarSiguienteId();
        mostrarResumenNegocio();
      } else {
        mensaje.textContent = `❌ No se encontró una reserva con ese ID.`;
      }
    }

    function verDisponibles() {
      const lista = document.getElementById("horariosDisponibles");
      lista.innerHTML = "";
      const ocupados = Object.values(reservas).map(r => r.horario);
      const disponibles = horarios.filter(h => !ocupados.includes(h));
      if (disponibles.length === 0) return lista.innerHTML = "<li>❌ No hay horarios disponibles</li>";
      disponibles.forEach(h => {
        const li = document.createElement("li");
        li.textContent = `🕒 ${h}`;
        lista.appendChild(li);
      });
    }

    function buscarComoCliente() {
      const id = document.getElementById("buscarCliente").value.trim();
      const resultado = document.getElementById("resultadoBusquedaCliente");
      if (reservas[id]) {
        resultado.textContent = `✅ Tu cita es a las ${reservas[id].horario}.`;
      } else {
        resultado.textContent = "❌ No se encontró tu cita.";
      }
    }

    function iniciarSesionDueño() {
      const clave = document.getElementById("claveDueño").value;
      const mensaje = document.getElementById("mensajeDueño");
      if (clave === CLAVE_DUEÑO) {
        document.getElementById("vistaDueño").style.display = "block";
        mensaje.textContent = "✅ Acceso concedido.";
      } else {
        mensaje.textContent = "❌ Contraseña incorrecta.";
      }
    }

    function verTodasLasReservas() {
      const lista = document.getElementById("reservasDueño");
      lista.innerHTML = "";
      if (Object.keys(reservas).length === 0) return lista.innerHTML = "<li>No hay reservas</li>";
      for (let id in reservas) {
        const li = document.createElement("li");
        li.textContent = `🧔 ID: ${id}, ${reservas[id].nombre} - ${reservas[id].horario}`;
        lista.appendChild(li);
      }
      mostrarResumenNegocio();
    }

    function mostrarResumenNegocio() {
      const total = Object.keys(reservas).length;
      const ocupados = total;
      const disponibles = horarios.length - total;
      const idsUsados = [...usados].join(", ") || "Ninguno";
      const idsLibres = 30 - usados.size;

      document.getElementById("resumenDueño").innerHTML = `
        <h3>📊 Estadísticas</h3>
        <p>🔢 Total de reservas: ${total}</p>
        <p>🧑‍🦱 Horarios ocupados: ${ocupados}</p>
        <p>🕒 Horarios disponibles: ${disponibles}</p>
        <p>🆔 IDs usados: ${idsUsados}</p>
        <p>🎯 IDs disponibles: ${idsLibres}</p>
        <p>❌ Cancelaciones: ${cancelacionesTotales}</p>
      `;
    }

    mostrarSiguienteId();
  </script>
</body>
</html
