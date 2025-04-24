<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Inicio de Sesión y Carta de Restaurante</title>
  <style>
    /* Estilos iguales que los tuyos, no los repito por brevedad */
  </style>
</head>
<body>

<div id="inicioSesion">
  <!-- Contenido de login -->
</div>

<div id="carta">
  <!-- Todo el contenido de la carta -->
</div>

<script>
  let inventario = JSON.parse(localStorage.getItem("inventario")) || [];
  let historialJefatura = JSON.parse(localStorage.getItem("historialJefatura")) || [];

  const tabla = document.querySelector("#tablaInventario tbody");
  const loginForm = document.getElementById('loginForm');
  const inicioSesionDiv = document.getElementById('inicioSesion');
  const cartaDiv = document.getElementById('carta');
  const infoUsuario = document.getElementById('infoUsuario');
  const formularioJefatura = document.getElementById('formularioJefatura');
  const ordenarPorSelect = document.getElementById('ordenarPor');
  const searchInput = document.getElementById('searchInput');

  function mostrarInformacionUsuario() {
    const nombre = localStorage.getItem('nombre');
    const rol = localStorage.getItem('rol');
    infoUsuario.innerText = `Bienvenido, ${nombre} (${rol})`;

    formularioJefatura.style.display = rol === 'jefatura' ? 'block' : 'none';
  }

  function actualizarTabla(orden = 'nombre', filtro = '') {
    tabla.innerHTML = "";
    let productosOrdenados = [...inventario];

    if (orden === 'nombre') {
      productosOrdenados.sort((a, b) => a.producto.localeCompare(b.producto));
    } else if (orden === 'categoria') {
      productosOrdenados.sort((a, b) => a.categoria.localeCompare(b.categoria));
    }

    if (filtro) {
      productosOrdenados = productosOrdenados.filter(item =>
        item.producto.toLowerCase().includes(filtro.toLowerCase())
      );
    }

    productosOrdenados.forEach((item, index) => {
      const fila = document.createElement("tr");
      const cantidadColor = item.cantidad < 20 ? '#e74c3c' : item.cantidad < 50 ? '#f39c12' : '#2ecc71';
      fila.innerHTML = `
        <td>${item.producto}</td>
        <td>${item.categoria}</td>
        <td style="background-color: ${cantidadColor};">${item.cantidad}</td>
        <td>${item.precio} €</td>
        <td>${item.persona}</td>
        <td>
          <button onclick="cambiarCantidad(${index}, 1)">+</button>
          <input type="number" value="${item.cantidad}" onchange="actualizarCantidad(${index}, this.value)" class="cantidad-input" />
          <button onclick="cambiarCantidad(${index}, -1)">-</button>
          <button onclick="guardarCambio(${index})">Guardar cambios</button>
          <button onclick="borrarProducto(${index})">Borrar</button>
        </td>
      `;
      tabla.appendChild(fila);
    });

    localStorage.setItem("inventario", JSON.stringify(inventario));
  }

  function cambiarCantidad(index, delta) {
    inventario[index].cantidad += delta;
    if (inventario[index].cantidad < 0) inventario[index].cantidad = 0;
    actualizarTabla();
  }

  function actualizarCantidad(index, nuevaCantidad) {
    inventario[index].cantidad = parseInt(nuevaCantidad, 10);
    if (inventario[index].cantidad < 0) inventario[index].cantidad = 0;
    actualizarTabla();
  }

  function guardarCambio(index) {
    inventario[index].persona = localStorage.getItem('nombre');
    actualizarTabla();
  }

  function borrarProducto(index) {
    if (confirm("¿Estás seguro de que deseas borrar este producto?")) {
      inventario.splice(index, 1);
      actualizarTabla();
    }
  }

  window.onload = function () {
    const nombre = localStorage.getItem('nombre');
    if (nombre) {
      inicioSesionDiv.style.display = 'none';
      cartaDiv.style.display = 'block';
      mostrarInformacionUsuario();
      actualizarTabla();
    } else {
      inicioSesionDiv.style.display = 'block';
      cartaDiv.style.display = 'none';
    }
  }

  loginForm.addEventListener('submit', function (event) {
    event.preventDefault();
    const nombre = document.getElementById('nombre').value;
    const codigo = document.getElementById('codigo').value;
    const rol = document.getElementById('rol').value;

    if (codigo === 'Dan') {
      localStorage.setItem('nombre', nombre);
      localStorage.setItem('rol', rol);
      inicioSesionDiv.style.display = 'none';
      cartaDiv.style.display = 'block';
      mostrarInformacionUsuario();
      actualizarTabla();
    } else {
      alert("Código incorrecto");
    }
  });

  ordenarPorSelect.addEventListener('change', function () {
    actualizarTabla(this.value);
  });

  document.getElementById('formularioProducto').addEventListener('submit', function (e) {
    e.preventDefault();
    const producto = document.getElementById('producto').value;
    const categoria = document.getElementById('categoria').value;
    const cantidad = parseInt(document.getElementById('cantidad').value, 10);
    const precio = parseFloat(document.getElementById('precio').value);
    const persona = localStorage.getItem('nombre');

    const nuevoProducto = { producto, categoria, cantidad, precio, persona };
    inventario.push(nuevoProducto);
    historialJefatura.push({ producto, cantidad, persona, fecha: new Date().toLocaleString() });

    localStorage.setItem("inventario", JSON.stringify(inventario));
    localStorage.setItem("historialJefatura", JSON.stringify(historialJefatura));
    actualizarTabla();
    e.target.reset();
  });

  searchInput.addEventListener('input', function () {
    actualizarTabla('nombre', this.value);
  });

  function cerrarSesion() {
    localStorage.removeItem('nombre');
    localStorage.removeItem('rol');
    inicioSesionDiv.style.display = 'block';
    cartaDiv.style.display = 'none';
  }
</script>

</body>
</html>
