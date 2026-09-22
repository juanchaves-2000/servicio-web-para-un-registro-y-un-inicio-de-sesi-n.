/**
 * @fileoverview Servicio Web (API REST) para Registro e Inicio de Sesión.
 * @author Juan Carlos Chaves Pérez
 * @description API desarrollada para la evidencia de "Construcción API". 
 * Incluye endpoints de registro y login con encriptación de contraseñas.
 */

const express = require('express');
const bcrypt = require('bcryptjs'); // Librería para encriptar contraseñas (Buena práctica)

const app = express();

// Middleware para permitir que la API reciba datos en formato JSON
app.use(express.json());

/**
 * Base de datos simulada en memoria.
 * Nota: En un entorno de producción, esto se reemplazaría por una conexión 
 * a una base de datos real (ej. MySQL, MongoDB, PostgreSQL).
 */
const usuariosDB = [];

/**
 * ENDPOINT 1: Registro de Usuarios
 * Ruta: POST /api/registro
 * Recibe: { "usuario": "...", "contrasena": "..." }
 */
app.post('/api/registro', async (req, res) => {
    try {
        const { usuario, contrasena } = req.body;

        // Validación de datos vacíos
        if (!usuario || !contrasena) {
            return res.status(400).json({ error: "El usuario y la contraseña son obligatorios." });
        }

        // Verificar si el usuario ya existe en nuestra base de datos simulada
        const existeUsuario = usuariosDB.find(u => u.usuario === usuario);
        if (existeUsuario) {
            return res.status(409).json({ error: "El usuario ya se encuentra registrado." });
        }

        // Encriptar la contraseña antes de guardarla (Estándar de seguridad)
        const salt = await bcrypt.genSalt(10);
        const contrasenaEncriptada = await bcrypt.hash(contrasena, salt);

        // Guardar el nuevo usuario
        usuariosDB.push({
            usuario: usuario,
            contrasena: contrasenaEncriptada
        });

        // Respuesta exitosa
        res.status(201).json({ mensaje: "Usuario registrado exitosamente." });
    } catch (error) {
        res.status(500).json({ error: "Error interno del servidor al registrar." });
    }
});

/**
 * ENDPOINT 2: Inicio de Sesión (Login)
 * Ruta: POST /api/login
 * Recibe: { "usuario": "...", "contrasena": "..." }
 */
app.post('/api/login', async (req, res) => {
    try {
        const { usuario, contrasena } = req.body;

        // 1. Buscar al usuario en la base de datos
        const userEncontrado = usuariosDB.find(u => u.usuario === usuario);

        // Si el usuario no existe, devuelve el error exigido por la guía
        if (!userEncontrado) {
            return res.status(401).json({ error: "Error en la autenticación." });
        }

        // 2. Comparar la contraseña en texto plano con la contraseña encriptada guardada
        const esContrasenaValida = await bcrypt.compare(contrasena, userEncontrado.contrasena);

        // 3. Validar el resultado y emitir los mensajes exactos de la guía
        if (esContrasenaValida) {
            res.status(200).json({ mensaje: "Autenticación satisfactoria." });
        } else {
            res.status(401).json({ error: "Error en la autenticación." });
        }
    } catch (error) {
        res.status(500).json({ error: "Error interno del servidor al iniciar sesión." });
    }
});

// Configuración del puerto y arranque del servidor
const PORT = 3000;
app.listen(PORT, () => {
    console.log(`Servidor API ejecutándose correctamente en http://localhost:${PORT}`);
});# servicio-web-para-un-registro-y-un-inicio-de-sesi-n.
Evidencia GA7-220501096-AA5-EV01. Diseño y desarrollo de servicios web - caso. 
