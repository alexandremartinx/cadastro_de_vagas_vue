<template>
  <link
    href="https://cdn.jsdelivr.net/npm/tailwindcss@2.1.2/dist/tailwind.min.css"
    rel="stylesheet"
  />

  <div class="relative">
    <div
      v-if="successMessage"
      class="fixed top-4 left-1/2 transform -translate-x-1/2 max-w-xl w-full mx-auto p-4 bg-green-100 text-green-700 border border-green-400 rounded shadow-lg z-50"
    >
      {{ successMessage }}
    </div>

    <h1 class="text-4xl font-bold text-center my-6">Lista de Vagas</h1>

    <div
      v-for="vaga in vagas"
      :key="vaga.id"
      class="max-w-5xl w-full m-2 justify-between mx-auto sm:px-6 lg:px-8 flex flex-row bg-white overflow-hidden shadow sm:rounded-lg p-6 items-center align-center"
    >
      <div>
        <h2 class="text-2xl leading-7 font-semibold">{{ vaga.titulo }}</h2>
        <p class="mt-1 max-w-md text-gray-600">{{ vaga.descricao }}</p>
      </div>

      <p class="text-gray-600">{{ vaga.beneficios }}</p>

      <div>
        <button
          type="button"
          @click="openModal(vaga)"
          class="inline-flex justify-center rounded-md border border-transparent bg-blue-100 px-4 py-2 text-sm font-medium text-blue-900 hover:bg-blue-200 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2"
        >
          Cadastrar
        </button>
      </div>
    </div>

    <TransitionRoot appear :show="isOpen" as="template">
      <Dialog as="div" @close="closeModal" class="relative z-10">
        <TransitionChild
          as="template"
          enter="duration-300 ease-out"
          enter-from="opacity-0"
          enter-to="opacity-100"
          leave="duration-200 ease-in"
          leave-from="opacity-100"
          leave-to="opacity-0"
        >
          <div class="fixed inset-0 bg-black bg-opacity-25" />
        </TransitionChild>

        <div class="fixed inset-0 overflow-y-auto">
          <div
            class="flex min-h-full items-center justify-center p-4 text-center"
          >
            <TransitionChild
              as="template"
              enter="duration-300 ease-out"
              enter-from="opacity-0 scale-95"
              enter-to="opacity-100 scale-100"
              leave="duration-200 ease-in"
              leave-from="opacity-100 scale-100"
              leave-to="opacity-0 scale-95"
            >
              <DialogPanel
                class="w-full max-w-md transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all"
              >
                <DialogTitle
                  as="h3"
                  class="text-lg font-medium leading-6 text-gray-900"
                >
                  Cadastrar na Vaga
                </DialogTitle>

                <form class="py-4" @submit.prevent="criaCandidato">
                  <label class="block py-2">
                    <span class="block text-sm font-medium text-slate-700"
                      >Nome</span
                    >
                    <input
                      id="nome"
                      name="nome"
                      v-model="nome"
                      required
                      class="border-zinc-500 border-2 rounded-md w-full p-2 placeholder-slate-400 contrast-more:border-slate-400 contrast-more:placeholder-slate-500"
                    />
                  </label>

                  <label class="block py-2">
                    <span class="block text-sm font-medium text-slate-700"
                      >E-mail</span
                    >
                    <input
                      id="email"
                      type="email"
                      name="email"
                      v-model="email"
                      required
                      class="border-zinc-500 border-2 w-full rounded-md p-2 placeholder-slate-400 contrast-more:border-slate-400 contrast-more:placeholder-slate-500"
                    />
                  </label>

                  <label class="block py-2">
                    <span class="block text-sm font-medium text-slate-700"
                      >Link do Currículo / Perfil do Linkedin</span
                    >
                    <input
                      id="curriculo"
                      name="curriculo"
                      v-model="curriculo"
                      required
                      class="border-zinc-500 border-2 w-full rounded-md p-2 placeholder-slate-400 contrast-more:border-slate-400 contrast-more:placeholder-slate-500"
                    />
                  </label>

                  <div class="mt-4">
                    <button
                      type="submit"
                      class="inline-flex justify-center rounded-md border border-transparent bg-blue-100 px-4 py-2 text-sm font-medium text-blue-900 hover:bg-blue-200 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2"
                    >
                      Cadastrar
                    </button>
                  </div>
                </form>
              </DialogPanel>
            </TransitionChild>
          </div>
        </div>
      </Dialog>
    </TransitionRoot>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import {
  TransitionRoot,
  TransitionChild,
  Dialog,
  DialogPanel,
  DialogTitle,
} from "@headlessui/vue";

const isOpen = ref(false);
const successMessage = ref("");
const vagas = ref([]);

function closeModal() {
  isOpen.value = false;
}
function openModal(vaga) {
  isOpen.value = true;
  // Aqui você pode definir a lógica para lidar com a vaga selecionada
  // Exemplo: Salvar a vaga selecionada em uma variável reativa
  selectedVaga.value = vaga;
}

const supabase = useSupabaseClient();

// Carregar dados das vagas
onMounted(async () => {
  const response = await supabase.from("vagas").select("*");
  if (response.error) {
    alert("Erro ao carregar vagas.");
  } else {
    vagas.value = response.data;
  }
});

const nome = ref("");
const email = ref("");
const curriculo = ref("");

const selectedVaga = ref(null); // Variável para armazenar a vaga selecionada

async function criaCandidato() {
  try {
    if (!selectedVaga.value) {
      throw new Error("Nenhuma vaga selecionada.");
    }

    const candidato = {
      nome: nome.value,
      email: email.value,
      curriculo: curriculo.value,
      id_vaga: selectedVaga.value.id, // Incluir o id da vaga selecionada
    };

    let { error } = await supabase.from("cadastros").insert(candidato);
    if (error) throw error;

    successMessage.value = "Cadastro realizado com sucesso!";

    // Limpar os campos do formulário
    nome.value = "";
    email.value = "";
    curriculo.value = "";

    // Limpar mensagem de sucesso após 5 segundos
    setTimeout(() => {
      successMessage.value = "";
    }, 5000); // 5000 milissegundos = 5 segundos
  } catch (error) {
    alert(error.message);
  }
}
</script>

