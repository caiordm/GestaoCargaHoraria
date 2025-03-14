<script setup>
import { shallowRef, markRaw, watch, onMounted } from "vue";
import { useRouter } from "vue-router";
import DisciplinaDetalhes from "@/components/Disciplinas/DisciplinaDetalhes.vue";
import DisciplinaItem from "./DisciplinaItem.vue";
import { ref, computed } from "vue";

const router = useRouter();
const currentComponent = shallowRef(null);
const componentProps = shallowRef({});
const disciplinas = ref([]);
const cursos = ref([]);

const fetchDisciplinas = async () => {
  try {
    const response = await fetch("http://localhost:5117/api/disciplina");
    const data = await response.json();
    disciplinas.value = data;
  } catch (error) {
    console.error("Erro ao buscar disciplinas:", error);
  }
};

const fetchCursos = async () => {
  try {
    const response = await fetch("http://localhost:5117/api/curso");
    const data = await response.json();
    cursos.value = data;
  } catch (error) {
    console.error("Erro ao buscar cursos:", error);
  }
};

onMounted(() => {
  fetchDisciplinas();
  fetchCursos();
});

const deactivateDetails = () => {
  currentComponent.value = null;
  componentProps.value = {};
};

const showComponent = (component, dis = {}) => {
  currentComponent.value = markRaw(component);
  componentProps.value = { ...dis };
};

const nextPage = (disciplinaId) => {
  console.log(disciplinaId);
  router.push("/Disciplinas/" + disciplinaId);
};

const searchQuery = ref("");
const filterProfessor = ref(false);
const filterCurso = ref("");

const filteredDisciplinas = computed(() => {
  return disciplinas.value.filter((disciplina) => {
    const matchesQuery = disciplina.name
      ? disciplina.name.toLowerCase().includes(searchQuery.value.toLowerCase())
      : false;
    const matchesProfessor = !filterProfessor.value || disciplina.docenteId === null;
    const matchesCurso =
      filterCurso.value === "" || disciplina.cursoId === filterCurso.value;
    return matchesQuery && matchesProfessor && matchesCurso;
  });
});

watch([searchQuery, filterProfessor, filterCurso], () => {
  deactivateDetails();
});
</script>

<template>
  <div class="container">
    <div class="filtro">
      <input type="text" v-model="searchQuery" @input="deactivateDetails" placeholder="Pesquisar por nome..." />
      <select v-model="filterCurso" @change="deactivateDetails">
        <option value="">Todos os Cursos</option>
        <option v-for="curso in cursos" :key="curso.id" :value="curso.id">
          {{ curso.name }}
        </option>
      </select>
      <label>
        <input type="checkbox" v-model="filterProfessor" @change="deactivateDetails" />
        Sem Professor
      </label>
    </div>
    <div class="container-disciplinas">

      <DisciplinaItem v-for="disciplina in filteredDisciplinas" :key="disciplina.id" :disciplina="disciplina"
        @click="showComponent(DisciplinaDetalhes, { disciplina })" />
    </div>
    <div class="painel">
      <component :is="currentComponent" v-bind="componentProps" />
      <button v-if="currentComponent" @click="nextPage(componentProps.disciplina?.id)">ver</button>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  width: 100%;
  height: 100vh;
  padding: 20px;
}

.container-disciplinas {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: start;
  background-color: transparent;
  width: 100%;
  height: 400px;
  gap: 5px;
  padding: 0px;
  overflow-x: auto;
}

.cards {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  width: 100%;
  gap: 20px;
  padding: 0px;
}

.painel {
  margin-top: 20px;
  width: 100%;
  height: 100%;
  background-color: transparent;
  position: relative;
}

.painel button {
  position: absolute;
  top: 10px;
  right: 10px;
  background-color: green;
  color: white;
  border-radius: 10px;
  border: 1px solid green;
  padding: 5px 10px;
  cursor: pointer;
}

.filtro {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.filtro input[type="text"] {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.filtro select {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-left: 10px;
}

.filtro button {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

.filtro button:hover {
  background-color: #0056b3;
}

.filtro label {
  display: flex;
  align-items: center;
  gap: 5px;
}
</style>
