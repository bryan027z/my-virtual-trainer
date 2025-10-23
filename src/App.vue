<template>
  <div class="min-h-screen bg-gray-50 text-gray-800">
    <header class="max-w-3xl mx-auto px-4 py-8">
      <h1 class="text-3xl font-bold flex items-center gap-2">
        🏋️ My Virtual Trainer
      </h1>
      <p class="text-sm text-gray-500">Registra, edita y visualiza tus entrenamientos en tiempo real.</p>
    </header>

    <main class="max-w-3xl mx-auto px-4 pb-16">
      <!-- Card: Formulario -->
      <section class="bg-white shadow rounded-2xl p-6 mb-8">
        <h2 class="text-lg font-semibold mb-4">
          {{ editandoId ? "Editar entrenamiento" : "Nuevo entrenamiento" }}
        </h2>

        <form @submit.prevent="guardar" class="grid grid-cols-1 sm:grid-cols-4 gap-3">
          <input type="date" v-model="form.fecha" required class="border rounded-lg px-3 py-2" />
          <input type="number" min="1" v-model.number="form.duracion" placeholder="Duración (min)" required class="border rounded-lg px-3 py-2" />
          <input type="number" min="0" step="0.01" v-model.number="form.distancia" placeholder="Distancia (km)" required class="border rounded-lg px-3 py-2" />

          <div class="flex gap-2">
            <button type="submit" class="flex-1 bg-blue-600 text-white rounded-lg px-3 py-2 hover:bg-blue-700">
              {{ editandoId ? "Actualizar" : "Guardar" }}
            </button>
            <button v-if="editandoId" type="button" @click="cancelar"
                    class="flex-1 bg-gray-200 rounded-lg px-3 py-2 hover:bg-gray-300">
              Cancelar
            </button>
          </div>
        </form>
      </section>

      <!-- Historial en tiempo real -->
      <section>
        <h2 class="text-lg font-semibold mb-4">Historial</h2>

        <div class="grid gap-4">
          <article v-for="t in entrenamientos" :key="t.id"
                   class="bg-white shadow rounded-2xl p-4 flex items-center justify-between">
            <div>
              <p class="font-semibold">{{ t.fecha }}</p>
              <p class="text-sm text-gray-500">{{ t.duracion }} min — {{ t.distancia }} km</p>
            </div>
            <div class="flex gap-2">
              <button @click="editar(t)" title="Editar"
                      class="px-3 py-1 rounded-lg border border-yellow-400 text-yellow-600 hover:bg-yellow-50">✏️</button>
              <button @click="eliminar(t.id!)" title="Eliminar"
                      class="px-3 py-1 rounded-lg border border-red-500 text-red-600 hover:bg-red-50">🗑️</button>
            </div>
          </article>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { db } from "./firebase";
import {
  collection, addDoc, onSnapshot, doc, deleteDoc, updateDoc,
  query, orderBy
} from "firebase/firestore";
import type { Entrenamiento } from "./types/Entrenamiento";

const form = ref<Entrenamiento>({ fecha: "", duracion: 0, distancia: 0 });
const entrenamientos = ref<Entrenamiento[]>([]);
const editandoId = ref<string | null>(null);

const colRef = collection(db, "entrenamientos");

// 🔴 LISTADO EN TIEMPO REAL + ORDENADO POR FECHA DESC
onMounted(() => {
  const q = query(colRef, orderBy("fecha", "desc"));
  onSnapshot(q, (snap) => {
    entrenamientos.value = snap.docs.map(d => ({ id: d.id, ...(d.data() as Omit<Entrenamiento,"id">) }));
  });
});

// ✅ GUARDAR / ACTUALIZAR
async function guardar() {
  if (!form.value.fecha || form.value.duracion <= 0 || form.value.distancia < 0) {
    alert("Completa datos válidos.");
    return;
  }
  if (editandoId.value) {
    await updateDoc(doc(db, "entrenamientos", editandoId.value), { ...form.value });
    editandoId.value = null;
  } else {
    await addDoc(colRef, { ...form.value });
  }
  limpiar();
}

// ✏️ CARGAR EN FORMULARIO
function editar(t: Entrenamiento) {
  form.value = { ...t };
  editandoId.value = t.id!;
}

// ❌ ELIMINAR
async function eliminar(id: string) {
  await deleteDoc(doc(db, "entrenamientos", id));
}

// ↩️ CANCELAR EDICIÓN
function cancelar() {
  limpiar();
  editandoId.value = null;
}

// 🧼 LIMPIAR
function limpiar() {
  form.value = { fecha: "", duracion: 0, distancia: 0 };
}
</script>

<style>
/* Si no instalaste Tailwind, este mínimo asegura tipografía limpia */
body { font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji"; }
</style>