<template>
  <header id="cabeceralogo" class="bg-light p-3 w-100">
    <h1>MOBILE STORE</h1>
</header>

<nav id="menuprincipal" class="navbar navbar-expand-lg navbar-light bg-light w-100">
    <ul class="navbar-nav justify-content-center">
      <li>
          <RouterLink to="/CarritoView" class="text-white nav-link">
            <i class="fas fa-database"></i> Carrito
          </RouterLink>
        </li>
    </ul>
</nav>
  <div id="productos">
    <div
      v-for="producto in productos"
      :key="producto.PRD_ID"
      :id="'producto-' + producto.PRD_NOMBRE"
      class="producto"
      :category="producto.PRD_MARCA"
    >
      <div class="foto-container">
        <div
          id="foto"
          :style="{ backgroundImage: `url(${producto.PRD_IMAGEN})`, height: '150px', backgroundSize: 'cover' }"
        ></div>
      </div>
      <div class="producto-info">
        <h2>{{ producto.PRD_NOMBRE }}</h2>
        <h3>Precio: {{ producto.PRD_PRECIO }}$</h3>
        <div class="producto-actions">
          <input
            v-model.number="producto.cantidad"
            type="number"
            placeholder="0"
            :min="0"
            :max="producto.PRD_STOCK"
          />
          <button
            :style="{ backgroundImage: 'url(https://res.cloudinary.com/dwp3lylw4/image/upload/v1717542230/carrito_i2wign.png)' }"
            @click="agregarAlCarrito(producto)"
          ></button>
        </div>
      </div>
    </div>
  </div>
  <footer id="pielogo" class="bg-light w-100">
    <div class="container-fluid">
        <div class="row">
            <section class="col-md-4">
                <h2>Todos los derechos reservados<br>Aguas Internacionales</h2>
            </section>
        </div>
    </div>
</footer>
</template>

<script>
export default {
  data() {
    return {
      productos: [],
    };
  },
  mounted() {
    this.obtenerProductos();
  },
  methods: {
    async obtenerProductos() {
      try {
        const response = await fetch('/api/Producto');
        if (!response.ok) {
          throw new Error('Error en la respuesta de la API');
        }
        const data = await response.json();
        this.productos = data.map(producto => ({
          ...producto,
          cantidad: 0, // Añadimos la propiedad cantidad
        }));
      } catch (error) {
        console.error('Error al obtener productos:', error);
      }
    },
    agregarAlCarrito(producto) {
      const cantidad = producto.cantidad;
      const maximo = producto.PRD_STOCK;
      const precio = producto.PRD_PRECIO;
      const id = producto.PRD_ID;

      if (cantidad > 0 && cantidad <= maximo) {
        let carrito = JSON.parse(localStorage.getItem('carrito')) || [];
        let item = carrito.find(p => p.id === id);
        if (item) {
          item.cantidad += cantidad;
        } else {
          carrito.push({
            nombre: producto.PRD_NOMBRE,
            cantidad,
            precio,
            id,
          });
        }
        localStorage.setItem('carrito', JSON.stringify(carrito));
        alert('Producto agregado al carrito');
        producto.cantidad = 0; // Limpiar el campo de cantidad después de agregar
      } else {
        alert('Cantidad no válida');
      }
    },
  },
};
</script>


<style>
/* Importar fuente de Google Fonts */
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap');

/* Estilos generales */
body {
    font-family: 'Roboto', sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f7fc; /* Fondo suave azul-grisáceo */
    color: #333;
}

/* Contenedor principal */
#menuYProductos {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

/* Estilos para el encabezado */
header#cabeceralogo {
    background: linear-gradient(135deg, #4a90e2, #6ab8f7); /* Degradado moderno */
    color: #ffffff;
    text-align: center;
    padding: 25px 0;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

header#cabeceralogo h1 {
    margin: 0;
    font-size: 2.8rem;
    font-weight: bold;
    letter-spacing: 1.5px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

/* Estilos para la barra de navegación */
nav#menuprincipal {
    background-color: #3a4a6c;
    padding: 10px 0;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

nav#menuprincipal .navbar-nav {
    display: flex;
    justify-content: center;
    gap: 20px;
}

nav#menuprincipal .nav-link {
    color: #ffffff;
    font-weight: 500;
    text-decoration: none;
    font-size: 1.1rem;
    padding: 8px 15px;
    border-radius: 20px;
    background-color: rgba(255, 255, 255, 0.15);
    transition: background-color 0.3s, transform 0.2s;
}

nav#menuprincipal .nav-link:hover {
    background-color: #7ec8e2;
    transform: scale(1.05);
}

nav#menuprincipal .navbar-nav {
    list-style: none; /* Elimina los puntos de la lista */
}

nav#menuprincipal .navbar-nav li {
    display: inline-block; /* Asegura que los elementos se alineen horizontalmente */
}

/* Contenedor principal de productos */
main.container-fluid {
    flex-grow: 1;
    padding: 40px 20px;
    background-color: #f4f7fc;
}

#productosActuales {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 30px;
}

/* Estilos para cada producto */
#productos {
    padding: 2%;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    border-radius: 10px;
}

    #productos > .producto {
        width: 240px;
        height: 340px;
        background-color: #ffffff;
        border: 1px solid #d0d8e0; /* Borde gris claro */
        border-radius: 12px;
        display: flex;
        flex-direction: column;
        overflow: hidden;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        text-align: center;
        padding: 15px;
        margin: 15px;
        background-color: #f7f9fc; /* Color de fondo más claro */
        transition: all 0.3s ease;
    }

        #productos > .producto:hover {
            transform: translateY(-10px);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
            background: linear-gradient(135deg, #c6e2f7, #a3c9e8); /* Degradado azul suave */
        }

/* Contenedor de la imagen */
.foto-container {
    width: 100%;
    height: 170px;
    margin-bottom: 15px;
}

/* Estilo para la imagen de fondo */
#productos > .producto #foto {
    width: 100%;
    height: 100%;
    background-size: cover;
    background-position: center;
    border-radius: 10px;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

/* Contenedor para la información del producto */
.producto-info {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    flex-grow: 1;
}

    .producto-info h2 {
        font-size: 18px;
        color: #333;
        margin: 0;
        font-weight: 600;
        line-height: 1.2;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        margin-bottom: 10px;
    }

    .producto-info h3 {
        font-size: 16px;
        color: #4a90e2; /* Azul más frío */
        margin: 10px 0;
        font-weight: normal;
        text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.2);
    }

/* Contenedor de las acciones (input y botón) */
.producto-actions {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 15px;
}

    /* Estilo para el campo de entrada de cantidad */
    .producto-actions input {
        width: 50%;
        padding: 10px;
        font-size: 16px;
        border: 1px solid #d0d8e0; /* Gris claro */
        border-radius: 8px;
        text-align: center;
        outline: none;
        background-color: rgba(255, 255, 255, 0.6);
        transition: background-color 0.3s ease;
    }

        .producto-actions input:focus {
            background-color: rgba(255, 255, 255, 1);
            border-color: #4a90e2; /* Azul frío en el borde */
        }

    /* Estilo para el botón de agregar al carrito */
    .producto-actions button {
        width: 40px;
        height: 40px;
        background-size: contain;
        background-repeat: no-repeat;
        border: none;
        cursor: pointer;
        background-color: transparent;
        transition: transform 0.3s ease, background-color 0.3s ease;
        border-radius: 8px;
    }

        /* Efecto hover en el botón */
        .producto-actions button:hover {
            transform: scale(1.1);
            background-color: rgba(255, 255, 255, 0.8);
        }

/* Pie de página */
footer#pielogo {
    background-color: #3a4a6c;
    padding: 20px 0;
    color: #d9e2f1;
    text-align: center;
}

footer#pielogo h2 {
    margin: 0;
    font-size: 1rem;
    font-weight: normal;
}

</style>