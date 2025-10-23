<template>
  <h1>🏋️ My Virtual Trainer</h1>

  <form @submit.prevent="guardar">
    <input type="date" v-model="fecha" required />
    <input type="number" v-model="duracion" placeholder="Duración (min)" required />
    <input type="number" v-model="distancia" placeholder="Distancia (km)" required />
    <button type="submit">{{ editando ? "Actualizar" : "Guardar" }}</button>
    <button type="button" v-if="editando" @click="cancelar">Cancelar</button>
  </form>

  <h2>Historial</h2>
  <ul>
    <li v-for="t in entrenamientos" :key="t.id">
      {{ t.fecha }} — {{ t.duracion }} min — {{ t.distancia }} km
      <button @click="editar(t)">✏️</button>
      <button @click="eliminar(t.id)">🗑️</button>
    </li>
  </ul>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { db } from "./firebase";
import { collection, addDoc, onSnapshot, deleteDoc, doc, updateDoc } from "firebase/firestore";

const fecha = ref("");
const duracion = ref<number | null>(null);
const distancia = ref<number | null>(null);
const entrenamientos = ref<any[]>([]);
const editando = ref(false);
let idActual: string | null = null;

const colRef = collection(db, "entrenamientos");

onMounted(() => {
  onSnapshot(colRef, (snap) => {
    entrenamientos.value = snap.docs.map((d) => ({ id: d.id, ...d.data() }));
  });
});

async function guardar() {
  if (editando.value && idActual) {
    await updateDoc(doc(db, "entrenamientos", idActual), {
      fecha: fecha.value,
      duracion: duracion.value,
      distancia: distancia.value,
    });
  } else {
    await addDoc(colRef, {
      fecha: fecha.value,
      duracion: duracion.value,
      distancia: distancia.value,
    });
  }
  limpiar();
}

function editar(t: any) {
  fecha.value = t.fecha;
  duracion.value = t.duracion;
  distancia.value = t.distancia;
  idActual = t.id;
  editando.value = true;
}

async function eliminar(id: string) {
  await deleteDoc(doc(db, "entrenamientos", id));
}

function cancelar() {
  limpiar();
}

function limpiar() {
  fecha.value = "";
  duracion.value = null;
  distancia.value = null;
  idActual = null;
  editando.value = false;
}
</script>