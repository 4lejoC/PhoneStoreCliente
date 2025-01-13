<template>
    <div>
        <!-- Login Form -->
        <div id="loginForm" class="container" v-if="!isLoggedIn">
            <h2>Login</h2>
            <h3>Ingrese su DNI</h3>
            <input type="text" v-model="DNI" class="form-control input-style" placeholder="DNI" required id="DNI">
            <h3>Ingrese el nombre de usuario</h3>
            <input type="text" v-model="username" class="form-control input-style" placeholder="usuario" required
                id="username">
            <h3>Ingrese la clave</h3>
            <input type="password" v-model="password" class="form-control input-style" placeholder="clave" required
                id="password">
            <button class="btn btn-primary btn-style" @click="login">Ingresar/Registrar</button>
            <br>
            <RouterLink to="/">
                <button class="btn btn-secondary btn-style">Volver</button>
            </RouterLink>
        </div>

        <!-- Página Principal -->
        <div id="paginaPrincipal" v-if="isLoggedIn">
            <header id="cabeceralogo" class="bg-light p-3 w-100">
                <div class="titulo">
                    <h1>MOBILE STORE</h1>
                </div>
                <button id="logout" class="btn btn-danger" @click="logout">Cerrar Sesion</button>
            </header>

            <nav id="menuprincipal" class="navbar navbar-expand-lg navbar-light bg-light w-100">
                <div class="container-fluid">
                    <div class="collapse navbar-collapse justify-content-center" id="navbarNav">
                        <ul class="navbar-nav justify-content-center">
                            <RouterLink to="/" class="text-white nav-link">
                                <i class="fas fa-database"></i> Volver
                            </RouterLink>
                        </ul>
                    </div>
                </div>
            </nav>

            <main>
                <section id="elementosCarrito" ref="elementosCarrito">
                    <!-- Elementos del carrito aquí -->
                </section>
                <aside>
                    <h2>Precio a pagar</h2>
                    <p>Subtotal: $<span>{{ subtotal.toFixed(2) }}</span></p>
                    <p>IVA (<span>{{ iva }}</span>%): $<span>{{ tiva.toFixed(2) }}</span></p>
                    <p>Total: $<span>{{ total.toFixed(2) }}</span></p>
                    <button @click="mostrarDatosFactura">Pagar</button>
                </aside>
            </main>

            <footer id="pielogo" class="bg-light w-100">
                <div class="container-fluid">
                    <div class="row">
                        <section class="col-md-4">
                            <h2>Todos los derechos reservados<br>Aguas Internacionales</h2>
                        </section>
                    </div>
                </div>
            </footer>
        </div>

        <!-- Modal -->
        <div id="modal" class="modal" v-if="isModalVisible">
            <div class="modal-content">
                <button class="close" @click="isModalVisible = false">X</button>
                <h2>Datos de la factura</h2>
                <label>Ingrese el correo:</label>
                <input type="text" v-model="correo" placeholder="Correo" required id="correo">
                <label>Ingrese la dirección:</label>
                <input type="text" v-model="direccion" placeholder="Dirección" required id="direccion">
                <label>Ingrese el número de teléfono:</label>
                <input type="number" v-model="telefono" placeholder="Teléfono" required id="telefono">
                <button @click="pagar">Pagar en Efectivo</button>
                <button @click="pagarDesdeBanca">Pagar desde la Banca</button>
            </div>
        </div>

    </div>
</template>

<script>
import axios from "axios";
export default {
    data() {
        return {
            DNI: "",
            username: "",
            password: "",
            isLoggedIn: false,
            isModalVisible: false,
            correo: "",
            direccion: "",
            telefono: "",
            subtotal: 0, // Inicializa con 0
            iva: 0,      // Inicializa con 0
            tiva: 0,     // Inicializa con 0
            total: 0,    // Inicializa con 0
        };
    },
    methods: {
        async login() {
            if (this.DNI && this.username && this.password) {
                try {
                    // Realizar la llamada al API
                    const response = await fetch(`/api/Usuario?DNI=${encodeURIComponent(this.DNI)}`);

                    if (!response.ok) {
                        throw new Error("Error al obtener el usuario");
                    }

                    const usuario = await response.json();

                    // Validar el usuario
                    if (usuario && usuario.USU_NOMBRE === this.username) {
                        if (usuario.USU_CONTRASENA === this.password) {
                            // Guardar cookies y actualizar el estado de la sesión
                            this.setCookie("dni", this.DNI);
                            this.setCookie("username", this.username);
                            this.isLoggedIn = true;
                        } else {
                            alert("Contraseña incorrecta.");
                        }
                    } else {
                        alert("Usuario no encontrado. Creando nuevo usuario.");
                        await this.crearUsuario(); // Llama a la función para crear un usuario
                    }
                } catch (error) {
                    console.error("Error al obtener usuario:", error);
                    alert("Error al intentar iniciar sesión.");
                }
            } else {
                alert("Por favor ingrese DNI, usuario y contraseña.");
            }
        },
        logout() {
            this.deleteCookie("username");
            this.deleteCookie("dni");
            this.isLoggedIn = false;
        },
        setCookie(name, value) {
            document.cookie = `${name}=${encodeURIComponent(value)}; path=/`;
        },
        deleteCookie(name) {
            document.cookie = `${name}=; path=/; expires=Thu, 01 Jan 1970 00:00:00 GMT`;
        },
        getCookie(name) {
            const cookieArray = document.cookie.split("; ");
            for (const cookie of cookieArray) {
                const [key, value] = cookie.split("=");
                if (key === name) return decodeURIComponent(value);
            }
            return null;
        },
        checkSession() {
            const dni = this.getCookie("dni");
            const username = this.getCookie("username");

            if (dni && username) {
                this.DNI = dni;
                this.username = username;
                this.isLoggedIn = true; // Actualizar estado
            } else {
                this.isLoggedIn = false; // Asegurar estado inicial
            }
        },

        async verCarrito() {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const contenedor = this.$refs.elementosCarrito;
            if (!contenedor) {
                return;
            }

            contenedor.innerHTML = ''; // Limpiar el contenedor

            if (carrito.length === 0) {
                contenedor.innerHTML = '<p>No hay productos en el carrito.</p>';
                return;
            }

            const tituloCarrito = document.createElement('h2');
            const botonBorrar = document.createElement('button');
            botonBorrar.textContent = 'Limpiar Carrito';
            botonBorrar.onclick = () => this.limpiarCarrito();
            tituloCarrito.textContent = 'Tu Carrito';
            contenedor.appendChild(tituloCarrito);
            contenedor.appendChild(botonBorrar);

            try {
                const response = await fetch('/api/Producto');
                if (!response.ok) throw new Error('Error al obtener productos desde la API');
                const productos = await response.json();

                carrito.forEach(item => {
                    const productoBaseDatos = productos.find(p => p.PRD_ID == item.id);
                    if (!productoBaseDatos) return; // Si no existe, saltar

                    const producto = document.createElement('div');
                    producto.classList.add('producto');

                    producto.innerHTML = `
                <div class="imagen-producto" style="background-image: url(${productoBaseDatos.PRD_IMAGEN});"></div>
                <div class="info-producto">
                    <h3>${productoBaseDatos.PRD_NOMBRE}</h3>
                    <p>Precio: $${productoBaseDatos.PRD_PRECIO}</p>
                </div>
                <div class="controles" data-id="${item.nombre}">
                    <button class="disminuir">-</button>
                    <input type="number" value="${item.cantidad}" readonly max="${productoBaseDatos.PRD_STOCK}">
                    <button class="aumentar">+</button>
                    <button class="eliminar">Eliminar</button>
                </div>
            `;
                    const eliminarButton = producto.querySelector('.eliminar');
                    eliminarButton.addEventListener('click', () => {
                        this.quitarDelCarrito(item.nombre);
                    });

                    const aumentarButton = producto.querySelector('.aumentar');
                    aumentarButton.addEventListener('click', () => {
                        this.aumentarProducto(item.nombre, productoBaseDatos.PRD_STOCK);
                    });

                    const disminuirButton = producto.querySelector('.disminuir');
                    disminuirButton.addEventListener('click', () => {
                        this.disminuirProducto(item.nombre);
                    });

                    contenedor.appendChild(producto);
                });
            } catch (error) {
                console.error('Error al procesar el carrito:', error);
                alert('No se pudieron cargar los productos.');
            }
        },

        quitarDelCarrito(nombreProducto) {
            let carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const index = carrito.findIndex(item => item.nombre === nombreProducto);
            if (index !== -1) {
                carrito.splice(index, 1);
                localStorage.setItem('carrito', JSON.stringify(carrito));
                this.verCarrito();
                this.calcularTotal();
            } else {
                console.log('El producto no se encontró en el carrito.');
            }
        },

        aumentarProducto(nombreProducto, maxStock) {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const index = carrito.findIndex(item => item.nombre === nombreProducto);

            if (index !== -1) {
                if (carrito[index].cantidad < maxStock) {
                    carrito[index].cantidad++;
                    localStorage.setItem('carrito', JSON.stringify(carrito));

                    // Actualiza el valor del input en el HTML
                    const inputCantidad = document.querySelector(`div[data-id='${nombreProducto}'] input[type='number']`);
                    if (inputCantidad) {
                        inputCantidad.value = carrito[index].cantidad;
                    }

                    this.calcularTotal();
                } else {
                    alert('Has alcanzado el límite de stock disponible para este producto.');
                }
            }
        },

        disminuirProducto(nombreProducto) {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const index = carrito.findIndex(item => item.nombre === nombreProducto);
            if (index !== -1) {
                carrito[index].cantidad--;
                if (carrito[index].cantidad > 0) {
                    localStorage.setItem('carrito', JSON.stringify(carrito));

                    // Actualiza el valor del input en el HTML
                    const inputCantidad = document.querySelector(`div[data-id='${nombreProducto}'] input[type='number']`);
                    if (inputCantidad) {
                        inputCantidad.value = carrito[index].cantidad;
                    }

                    this.calcularTotal();
                } else {
                    this.quitarDelCarrito(nombreProducto);
                }
            }
        },

        limpiarCarrito() {
            localStorage.removeItem('carrito');
            this.verCarrito();
            this.calcularTotal();
        },

        async calcularTotal() {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const id = 1; // El ID que se utiliza según tu código en el controlador

            try {
                const response = await fetch(`/api/DatosGenerales/${id}`);
                if (!response.ok) {
                    throw new Error('Error al obtener los datos generales');
                }

                const datosGenerales = await response.json();
                if (datosGenerales) {
                    const iva = datosGenerales.IVA;
                    const subtotal = carrito.reduce((sum, item) => sum + item.precio * item.cantidad, 0);
                    const tiva = subtotal * (iva / 100);
                    const total = subtotal + tiva;

                    // Actualiza las propiedades del estado
                    this.subtotal = subtotal;
                    this.iva = iva;
                    this.tiva = tiva;
                    this.total = total;
                } else {
                    console.error('No se encontraron datos generales.');
                }
            } catch (error) {
                console.error('Error:', error);
            }
        },

        mostrarDatosFactura() {
            const username = this.getCookie("username");

            if (username) {
                const carrito = JSON.parse(localStorage.getItem("carrito")) || [];
                if (carrito.length > 0) {
                    this.isModalVisible = true;
                } else {
                    alert("No hay objetos en el carrito.");
                }
            } else {
                alert("Por favor inicie sesión para continuar con el pago.");
            }
        },

        async pagarDesdeBanca() {
            const username = this.getCookie("username");
            const dni = this.getCookie("dni");
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];

            if (!username) {
                alert('Por favor inicie sesión para continuar con el pago.');
                return;
            }

            if (carrito.length === 0) {
                alert('No hay objetos en el carrito');
                return;
            }

            try {
                // Calcular el total de la factura
                const iva = this.iva;
                const subtotal = this.subtotal;
                const tiva = this.tiva;
                const totalFac = this.total;
                const id = 1;

                const response = await fetch(`/api/DatosGenerales/${id}`);
                if (!response.ok) {
                    throw new Error('Error al obtener los datos generales');
                }
                const datosGenerales = await response.json();

                // Verificar que la propiedad NUM_FAC existe
                if (!datosGenerales.NUM_FAC) {
                    throw new Error('Número de factura no disponible');
                }

                // Obtener datos adicionales para la factura
                const correo = document.getElementById('correo').value;
                const direcFac = document.getElementById('direccion').value;
                const telfFac = document.getElementById('telefono').value;
                const fechFac = new Date().toISOString();
                const facActual = "FAC" + datosGenerales.NUM_FAC;

                // Crear objeto de factura que coincida con el modelo FACTURA en la API
                const factura = {
                    FAC_NUMERO: facActual,
                    USU_DNI: dni,
                    USU_NOMBRE: username,
                    USU_CORREO: correo,
                    FAC_DIRECCION: direcFac,
                    FAC_TELEFONO: telfFac,
                    FAC_FECHA: fechFac,
                    FAC_TOTAL: totalFac,
                    FAC_ESTADO: "Pendiente"
                };

                // Guardar la factura en el servidor llamando al controlador Factura/Create
                const saveResponse = await axios.post(`/api/Factura`, factura);

                // No es necesario verificar saveResponse.status ya que Axios lanzará un error si hay un problema
                alert('Factura guardada correctamente.');

                // Llamadas a funciones adicionales para completar el proceso de pago
                this.agregarDetalleFactura(); // Si es necesario
                this.aumentarNumeroFactura(); // Si es necesario
                this.limpiarCarrito();

                // Limpiar campos del formulario y cerrar modal
                document.getElementById('modal').style.display = 'none';
                document.getElementById('direccion').value = '';
                document.getElementById('telefono').value = '';
                document.getElementById('correo').value = '';

            } catch (error) {
                console.error('Error al procesar el pago:', error);
                alert('Hubo un error al procesar el pago.');
            }
        },

        async pagar() {
            const username = this.getCookie("username");
            const dni = this.getCookie("dni");
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];

            if (!username) {
                alert('Por favor inicie sesión para continuar con el pago.');
                return;
            }

            if (carrito.length === 0) {
                alert('No hay objetos en el carrito');
                return;
            }

            try {
                // Calcular el total de la factura
                const iva = this.iva;
                const subtotal = this.subtotal;
                const tiva = this.tiva;
                const totalFac = this.total;
                const id = 1;

                const response = await fetch(`/api/DatosGenerales/${id}`);
                if (!response.ok) {
                    throw new Error('Error al obtener los datos generales');
                }
                const datosGenerales = await response.json();

                // Verificar que la propiedad NUM_FAC existe
                if (!datosGenerales.NUM_FAC) {
                    throw new Error('Número de factura no disponible');
                }

                // Obtener datos adicionales para la factura
                const correo = document.getElementById('correo').value;
                const direcFac = document.getElementById('direccion').value;
                const telfFac = document.getElementById('telefono').value;
                const fechFac = new Date().toISOString();
                const facActual = "FAC" + datosGenerales.NUM_FAC;

                // Crear objeto de factura que coincida con el modelo FACTURA en la API
                const factura = {
                    FAC_NUMERO: facActual,
                    USU_DNI: dni,
                    USU_NOMBRE: username,
                    USU_CORREO: correo,
                    FAC_DIRECCION: direcFac,
                    FAC_TELEFONO: telfFac,
                    FAC_FECHA: fechFac,
                    FAC_TOTAL: totalFac,
                    FAC_ESTADO: "Pagado"
                };

                // Guardar la factura en el servidor llamando al controlador Factura/Create
                const saveResponse = await axios.post(`/api/Factura`, factura);

                // No es necesario verificar saveResponse.status ya que Axios lanzará un error si hay un problema
                alert('Factura guardada correctamente.');

                // Llamadas a funciones adicionales para completar el proceso de pago
                this.agregarDetalleFactura();
                this.debitarProductos();
                this.aumentarNumeroFactura(); // Si es necesario
                this.limpiarCarrito();

                // Limpiar campos del formulario y cerrar modal
                document.getElementById('modal').style.display = 'none';
                document.getElementById('direccion').value = '';
                document.getElementById('telefono').value = '';
                document.getElementById('correo').value = '';

            } catch (error) {
                console.error('Error al procesar el pago:', error);
                alert('Hubo un error al procesar el pago.');
            }
        },

        async agregarDetalleFactura() {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];

            try {
                // Obtener datos generales como IVA
                const id = 1;
                const response = await fetch(`/api/DatosGenerales/${id}`);
                if (!response.ok) {
                    throw new Error('Error al obtener los datos generales');
                }
                const datosGenerales = await response.json();
                const iva = datosGenerales.IVA;

                // Generar el número de factura actual (ej. "FAC10")
                const facNumero = "FAC" + datosGenerales.NUM_FAC;

                // Guardar cada detalle de producto en la API
                for (const producto of carrito) {
                    try {
                        const subtotalPrd = (producto.precio * producto.cantidad) * (1 + (iva / 100));

                        const detalleFactura = {
                            FAC_NUMERO: facNumero,
                            PRD_ID: producto.id,
                            PRD_CANTIDAD: producto.cantidad,
                            PRD_SUBTOTAL: subtotalPrd
                        };

                        const detalleResponse = await axios.post(`/api/Detalle_Factura`, detalleFactura);

                        // Manejar la respuesta si es necesario
                        console.log('Detalle guardado:', detalleResponse.data);

                    } catch (error) {
                        console.error('Error al guardar detalle para el producto:', producto, error);
                        continue; // Continúa con el siguiente producto en el carrito
                    }
                }

                alert('Detalles de factura guardados correctamente.');
            } catch (error) {
                console.error('Error al guardar los detalles de factura:', error);
                alert('Hubo un error general al guardar los detalles de factura.');
            }
        },

        async debitarProductos() {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const errores = []; // Array para almacenar errores

            for (const producto of carrito) {
                try {
                    const response = await axios.get(`/api/Producto?PrdNombre=${producto.id}`);
                    const productoADebitar = response.data;

                    // Validar que el producto exista
                    if (!productoADebitar || typeof productoADebitar.PRD_STOCK === 'undefined') {
                        throw new Error(`Producto con ID ${producto.PRD_ID} no encontrado o sin stock disponible.`);
                    }

                    productoADebitar.PRD_STOCK = productoADebitar.PRD_STOCK - producto.cantidad;
                    if (productoADebitar.PRD_STOCK < 0) {
                        throw new Error(`Stock insuficiente para el producto ${productoADebitar.PRD_NOMBRE}.`);
                    }


                    // Enviar la actualización al servidor
                    const debitarResponse = await axios.put(`/api/Producto/${producto.PRD_ID}`, productoADebitar);

                } catch (error) {
                    // Manejar errores en la solicitud
                    const errorMessage = error.response?.data?.message || 'No se pudo debitar el producto';
                    console.error(`Error al procesar el producto ${producto.nombre}:`, errorMessage);
                    errores.push(`Error con el producto ID ${producto.PRD_ID}: ${errorMessage}`);
                }
            }

            // Muestra los errores al final, si existen
            if (errores.length > 0) {
                alert(`Se produjeron los siguientes errores:\n- ${errores.join('\n- ')}`);
            }
        },

        async aumentarNumeroFactura() {
            try {
                const id = 1;

                // Obtener los datos generales
                const response = await axios.get(`/api/DatosGenerales/${id}`);
                const datos = response.data;

                // Incrementar el número de factura
                datos.NUM_FAC++;

                // Enviar la actualización al servidor
                const CambiarResponse = await axios.put(`/api/DatosGenerales/${id}`, datos);

                console.log("Número de factura actualizado correctamente:", CambiarResponse.data);
            } catch (error) {
                const errorMessage = error.response?.data?.message || 'No se pudo actualizar el número de factura.';
                console.error("Error al actualizar el número de factura:", errorMessage);
                alert(`Error: ${errorMessage}`);
            }
        },

        async crearUsuario() {
            try {
                const nuevoDNI = document.getElementById("DNI").value;
                const nuevoUsuario = document.getElementById("username").value;
                const nuevaContrasena = document.getElementById("password").value;

                // Enviar la solicitud POST con Axios
                const response = await axios.post('/api/Usuario', {
                    USU_DNI: nuevoDNI,
                    USU_NOMBRE: nuevoUsuario,
                    USU_CONTRASENA: nuevaContrasena
                });

                // Verificar la respuesta
                if (response.status === 201 || response.status === 200) {
                    alert('Usuario creado exitosamente.');
                    console.log('Usuario creado:', response.data);
                } else {
                    console.error('Error al ingresar el nuevo usuario:', response);
                    alert('Hubo un error al ingresar el nuevo usuario.');
                }
            } catch (error) {
                // Manejar errores
                const errorMessage = error.response?.data?.message || 'Error desconocido.';
                console.error('Error al ingresar el nuevo usuario:', errorMessage);
                alert(`Hubo un error al ingresar el nuevo usuario: ${errorMessage}`);
            }
        }


    },
    mounted() {
        this.$nextTick(() => {
            this.checkSession();
            if (this.isLoggedIn) {
                setTimeout(() => {
                    this.verCarrito(); // Asegurar que el DOM esté listo
                }, 100); // Ajusta el tiempo si es necesario
            }
            this.calcularTotal();
        });
    }

};
</script>

<style>

/* Contenedor principal */
#loginForm {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    max-width: 400px;
    margin: 50px auto;
    padding: 20px;
    background-color: #f8f9fa;
    border: 1px solid #dee2e6;
    border-radius: 10px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

/* Encabezados */
#loginForm h2 {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 20px;
    color: #343a40;
    text-align: center;
}

#loginForm h3 {
    font-size: 16px;
    font-weight: 600;
    margin: 10px 0 5px;
    color: #495057;
}

/* Estilo de los inputs */
.input-style {
    width: 100%;
    padding: 10px 15px;
    margin-bottom: 15px;
    font-size: 14px;
    border: 1px solid #ced4da;
    border-radius: 5px;
    box-sizing: border-box;
    transition: border-color 0.3s ease;
}

.input-style:focus {
    outline: none;
    border-color: #007bff;
    box-shadow: 0 0 5px rgba(0, 123, 255, 0.25);
}

/* Estilo del botón */
.btn-style {
    width: 100%;
    padding: 10px 15px;
    font-size: 16px;
    font-weight: bold;
    color: #fff;
    background-color: #007bff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

.btn-style:hover {
    background-color: #0056b3;
}

.btn-style:active {
    background-color: #004085;
}

/* Adaptabilidad móvil */
@media (max-width: 768px) {
    #loginForm {
        padding: 15px;
        width: 90%;
    }

    .btn-style {
        font-size: 14px;
        padding: 8px 12px;
    }
}


/* Estilos generales */
body {
    font-family: Arial, sans-serif;
    color: #333;
    background-color: #f0f8ff;
    /* Fondo azul claro */
    margin: 0;
    padding: 0;
}

/* Estilos para los productos del carrito */
#elementosCarrito {
    display: flex;
    flex-direction: column;
    gap: 20px;
    margin-right: 15px;
    padding: 20px;
    background-color: #ffffe0;
    /* Fondo amarillo claro */
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.producto {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background-color: #fff;
    padding: 15px;
    border-radius: 8px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.producto-info {
    flex: 1;
    padding-left: 15px;
}

.imagen-producto {
    width: 100px;
    height: 100px;
    background-size: cover;
    background-position: center;
    border-radius: 8px;
}

.info-producto h3 {
    font-size: 1.2rem;
    color: #333;
    margin: 0;
}

.info-producto p {
    font-size: 1rem;
    color: #ff6347;
    margin-top: 5px;
}

/* Controles del carrito */
.controles {
    display: flex;
    align-items: center;
    gap: 10px;
}

.controles button,
.controles input {
    padding: 8px;
    font-size: 1rem;
    border-radius: 5px;
}

.controles button {
    background-color: #ff6347;
    color: white;
    border: none;
    cursor: pointer;
}

.controles button:hover {
    background-color: #ff4500;
}

.controles input {
    width: 50px;
    text-align: center;
    border: 1px solid #ddd;
    background-color: #f0f8ff;
}

.botonEliminar {
    background-color: #dc3545;
    color: white;
}

.botonEliminar:hover {
    background-color: #c82333;
}

/* Estilo para el botón limpiar carrito */
button {
    background-color: #28a745;
    color: white;
    padding: 10px;
    border-radius: 5px;
    cursor: pointer;
}

button:hover {
    background-color: #218838;
}

/* Modal */
.modal {
    position: fixed;
    z-index: 1000;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.6);
    /* Fondo más oscuro */
    display: flex;
    justify-content: center;
    align-items: center;
    animation: fadeIn 0.3s ease-in-out;
    /* Animación al aparecer */
}

.modal-content {
    background-color: #f9f9f9;
    /* Fondo más claro */
    margin: auto;
    padding: 30px;
    border: 1px solid #ddd;
    width: 90%;
    max-width: 500px;
    border-radius: 12px;
    /* Bordes redondeados */
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.3);
    /* Sombra más prominente */
    animation: slideDown 0.4s ease;
    /* Animación de deslizamiento */
    font-family: 'Arial', sans-serif;
    /* Fuente elegante y legible */
}

.modal-content h2 {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 20px;
    color: #333;
    text-align: center;
}

.modal-content label {
    display: block;
    margin-bottom: 10px;
    font-weight: 500;
    color: #555;
}

.modal-content input {
    width: calc(100% - 20px);
    padding: 10px;
    margin-bottom: 20px;
    border: 1px solid #ddd;
    border-radius: 6px;
    box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
    font-size: 14px;
}

.modal-content button {
    display: inline-block;
    width: 100%;
    padding: 12px;
    margin-top: 10px;
    color: white;
    font-size: 16px;
    font-weight: bold;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

.modal-content .close {
    border: none;
    font-size: 20px;
    font-weight: bold;
    cursor: pointer;
    transition: color 0.2s ease;
    width: 50px;
}

.modal-content .close:hover {
    color: red;
    /* Efecto al pasar el ratón */
}

/* Animaciones */
@keyframes fadeIn {
    from {
        opacity: 0;
    }

    to {
        opacity: 1;
    }
}

@keyframes slideDown {
    from {
        transform: translateY(-50px);
        opacity: 0;
    }

    to {
        transform: translateY(0);
        opacity: 1;
    }
}


/* Footer */
#pielogo {
    background-color: #333;
    color: white;
    text-align: center;
    padding: 20px 0;
}

#pielogo h2 {
    margin: 0;
    font-size: 1rem;
}
</style>