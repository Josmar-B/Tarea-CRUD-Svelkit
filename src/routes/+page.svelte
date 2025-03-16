<script>
    import { fade } from 'svelte/transition';

    let pacientes = $state([
        { nombre: 'John', apellido: 'Doe', descripcion: 'Paciente con dolor de cabeza', enfermedad: 'Migraña' },
        { nombre: 'Jane', apellido: 'Smith', descripcion: 'Paciente con fiebre alta', enfermedad: 'Gripe' },
        { nombre: 'Emily', apellido: 'Johnson', descripcion: 'Paciente con dolor abdominal', enfermedad: 'Gastritis' }
    ]);

    let prefijo = $state('');
    let nombre = $state('');
    let apellido = $state('');
    let descripcion = $state('');
    let enfermedad = $state('');
    let i = $state(0);

    let pacientesFiltrados = $derived(prefijo
        ? pacientes.filter((paciente) => {
                const nombreCompleto = `${paciente.apellido}, ${paciente.nombre}`;
                return nombreCompleto.toLowerCase().startsWith(prefijo.toLowerCase());
            })
        : pacientes);
    let seleccionado = $derived(pacientesFiltrados[i]);

    $effect(() => {
        reset_inputs(seleccionado);
    });

    // Notificaciones
    let notificacionVisible = $state(false);
    let notificacionMensaje = $state('');

    function mostrarNotificacion(mensaje) {
        notificacionMensaje = mensaje;
        notificacionVisible = true;
        setTimeout(() => (notificacionVisible = false), 2000); // Oculta la notificación después de 2 segundos
    }

    function crear() {
        pacientes = pacientes.concat({ nombre, apellido, descripcion, enfermedad });
        i = pacientes.length - 1;
        nombre = apellido = descripcion = enfermedad = '';
        mostrarNotificacion('¡Paciente creado con éxito!');
    }

    function actualizar() {
        seleccionado.nombre = nombre;
        seleccionado.apellido = apellido;
        seleccionado.descripcion = descripcion;
        seleccionado.enfermedad = enfermedad;
        pacientes = pacientes;
        mostrarNotificacion('¡Paciente actualizado con éxito!');
    }

    function eliminar() {
        const index = pacientes.indexOf(seleccionado);
        pacientes = [...pacientes.slice(0, index), ...pacientes.slice(index + 1)];

        nombre = apellido = descripcion = enfermedad = '';
        i = Math.min(i, pacientesFiltrados.length - 2);
        mostrarNotificacion('¡Paciente eliminado con éxito!');
    }

    function reset_inputs(paciente) {
        nombre = paciente ? paciente.nombre : '';
        apellido = paciente ? paciente.apellido : '';
        descripcion = paciente ? paciente.descripcion : '';
        enfermedad = paciente ? paciente.enfermedad : '';
    }
</script>

<div class="container">
    <div class="filtro">
        <input placeholder="Filtrar por apellido, nombre" bind:value={prefijo} />
    </div>

    <div class="lista">
        <select bind:value={i} size={5}>
            {#each pacientesFiltrados as paciente, i}
                <option value={i}>{paciente.apellido}, {paciente.nombre}</option>
            {/each}
        </select>
    </div>

    <div class="formulario">
        <label><input bind:value={nombre} placeholder="Nombre" /></label>
        <label><input bind:value={apellido} placeholder="Apellido" /></label>
        <label><input bind:value={descripcion} placeholder="Descripción" /></label>
        <label><input bind:value={enfermedad} placeholder="Enfermedad" /></label>

        <div class="buttons">
            <!-- Botón de Crear -->
            <button
                class="rotate-button"
                onclick={crear}
                disabled={!nombre || !apellido || !descripcion || !enfermedad}
            >
                Crear
            </button>

            <!-- Botón de Actualizar -->
            <button
                class="rotate-button"
                onclick={actualizar}
                disabled={!nombre || !apellido || !descripcion || !enfermedad || !seleccionado}
            >
                Actualizar
            </button>

            <!-- Botón de Eliminar -->
            <button
                class="rotate-button"
                onclick={eliminar}
                disabled={!seleccionado}
            >
                Eliminar
            </button>
        </div>

        <!-- Notificación -->
        {#if notificacionVisible}
            <div transition:fade class="notificacion">{notificacionMensaje}</div>
        {/if}
    </div>

    <div class="detalle">
        {#if seleccionado}
            <h3>Detalles del Paciente</h3>
            <p><strong>Nombre:</strong> {seleccionado.nombre} {seleccionado.apellido}</p>
            <p><strong>Descripción:</strong> {seleccionado.descripcion}</p>
            <p><strong>Enfermedad:</strong> {seleccionado.enfermedad}</p>
        {:else}
            <p>No hay paciente seleccionado.</p>
        {/if}
    </div>
</div>

<style>
    * {
        font-family: 'Arial', sans-serif;
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    .container {
        max-width: 800px;
        margin: 20px auto;
        padding: 20px;
        background-color: #f9f9f9;
        border-radius: 10px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .filtro input {
        width: 100%;
        padding: 10px;
        margin-bottom: 20px;
        border: 1px solid #ccc;
        border-radius: 5px;
        font-size: 16px;
    }

    .lista select {
        width: 100%;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
        font-size: 16px;
        background-color: #fff;
    }

    .formulario {
        margin-top: 20px;
    }

    .formulario label {
        display: block;
        margin-bottom: 10px;
    }

    .formulario input {
        width: 100%;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
        font-size: 16px;
    }

    .buttons {
        margin-top: 20px;
        display: flex;
        gap: 10px;
    }

    .buttons button {
        padding: 10px 20px;
        border: none;
        border-radius: 5px;
        background-color: #007bff;
        color: white;
        font-size: 16px;
        cursor: pointer;
        transition: background-color 0.3s, transform 0.5s;
    }

    .buttons button:disabled {
        background-color: #ccc;
        cursor: not-allowed;
    }

    .buttons button:hover:not(:disabled) {
        background-color: #0056b3;
        animation: rotate 1s linear infinite;
    }

    @keyframes rotate {
        from {
            transform: rotate(0deg);
        }
        to {
            transform: rotate(360deg);
        }
    }

    .detalle {
        margin-top: 20px;
        padding: 20px;
        background-color: #fff;
        border: 1px solid #ccc;
        border-radius: 5px;
    }

    .detalle h3 {
        margin-bottom: 10px;
        font-size: 20px;
        color: #333;
    }

    .detalle p {
        margin-bottom: 10px;
        font-size: 16px;
        color: #555;
    }

    /* Estilos para la notificación */
    .notificacion {
        margin-top: 10px;
        padding: 10px;
        background-color: #4caf50;
        color: white;
        border-radius: 5px;
        text-align: center;
        font-size: 14px;
    }
</style>