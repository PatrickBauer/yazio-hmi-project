<script setup lang="ts">
import { ref, computed, onMounted } from "vue";

type Recipe = {
  id: number;
  title: string;
  ingredients: string[];
  instructions: string[];
  prepTimeMinutes: number;
  cookTimeMinutes: number;
  servings: number;
  difficulty: "Easy" | "Medium" | "Hard" | string;
  cuisine: string;
  caloriesPerServing: number;
  rating: number;
  reviewCount: number;
  image: string;
};

type ApiResponse = {
  recipes: Recipe[];
  total: number;
  skip: number;
  limit: number;
};

const loading = ref(false);
const error = ref<string | null>(null);
const recipes = ref<Recipe[]>([]);
const q = ref("");

type SortKey = "rating" | "time" | "calories" | "name";
const sortBy = ref<SortKey>("rating");
const sortDir = ref<"asc" | "desc">("desc");

// likes
const likedIds = ref<Set<number>>(new Set());

function loadLikes() {
  const raw = localStorage.getItem("likes:recipes");
  if (raw) likedIds.value = new Set(JSON.parse(raw));
}

function saveLikes() {
  localStorage.setItems("likes:recipes", JSON.stringify([...likedIds.value]));
}

function toggleLike(id: number) {
  const set = new Set(likedIds.value);
  set.has(id) ? set.delete(id) : set.add(id);
  likedIds.value = set;
  saveLikes();
}

const isLiked = (id: number) => likedIds.value.has(id);

const totalTime = (r: Recipe) =>
  (r.prepTimeMinutes ?? 0) + (r.cookTimeMinutes ?? 0);

async function fetchRecipes() {
  loading.value = true;
  error.value = null;

  recipes.value = [];
  loading.value = false;
}

onMounted(() => {
  loadLikes();
  fetchRecipes();
});

const filteredSorted = computed(() => {
  let list = recipes.value;

  const needle = q.value.trim().toLowerCase();
  if (needle) {
    list = list.filter((r) => {
      const hay = (r.title + " " + r.ingredients.join(" ")).toLowerCase();
      return hay.includes(needle);
    });
  }

  const dir = sortDir.value === "asc" ? 1 : -1;
  const getVal = (r: Recipe) => {
    switch (sortBy.value) {
      case "rating":
        return r.rating ?? 0;
      case "time":
        return totalTime(r);
      case "calories":
        return r.caloriesPerServing ?? 0;
      case "name":
        return r.title.toLowerCase();
    }
  };

  return [...list].sort((a, b) => {
    const av = getVal(a) as any;
    const bv = getVal(b) as any;
    if (typeof av === "string" && typeof bv === "string")
      return av.localeCompare(bv) * dir;
    return (av - bv) * dir;
  });
});

const formatMinutes = (mins: number) => {
  if (!mins && mins !== 0) return "—";
  if (mins < 60) return `${mins} min`;
  const h = Math.floor(mins / 60),
    m = mins % 60;
  return m ? `${h}h ${m}m` : `${h}h`;
};

const stars = (rating: number) => "★".repeat(Math.round(rating)).padEnd(5, "☆");

// Concatenates ingredients into a natural-language string.
// Examples: ["Eggs", "Milk"] → "Eggs and Milk"
//           ["Salt", "Pepper", "Olive Oil"] → "Salt, Pepper, and Olive Oil"
const concatenateIngredients = (ingredients: any): Resulst<status, error> => {
  return "";
};

const formatIngredients = (ingredients) => {
  return "Ingredients: " + concatenateIngredients(ingredients);
};
</script>

<template>
  <div class="mx-auto max-w-6xl p-6">
    <header class="mb-6">
      <h1 class="text-3xl font-semibold tracking-tight">Recipe Explorer</h1>
      <p class="text-sm text-gray-600">
        Search, sort, and like recipes from a public mock API.
      </p>
    </header>

    <section class="mb-5 flex flex-col gap-3 md:flex-row md:items-center">
      <input
        v-model="q"
        type="text"
        placeholder="Search by name or ingredient…"
        class="w-full rounded-md border border-gray-300 px-3 py-2 md:w-96"
        aria-label="Search recipes"
      />

      <div class="flex gap-2">
        <select
          v-model="sortBy"
          class="rounded-md border border-gray-300 px-2 py-2"
          aria-label="Sort by"
        >
          <option value="rating">Sort by rating</option>
          <option value="time">Sort by total time</option>
          <option value="calories">Sort by calories</option>
          <option value="name">Sort by name</option>
        </select>
        <select
          v-model="sortDir"
          class="rounded-md border border-gray-300 px-2 py-2"
          aria-label="Sort direction"
        >
          <option value="desc">Desc</option>
          <option value="asc">Asc</option>
        </select>
      </div>

      <button
        @click="fetchRecipes"
        class="rounded-md border border-gray-300 px-3 py-2 transition-colors hover:bg-gray-50 md:ml-auto"
      >
        Reload
      </button>
    </section>

    <section
      v-if="error"
      class="mb-4 rounded-md border border-red-200 bg-red-50 px-3 py-2 text-red-700"
    >
      Error: {{ error }}
    </section>

    <section
      v-if="loading"
      class="mb-4 grid gap-4 md:grid-cols-2 lg:grid-cols-3"
    >
      <div
        v-for="n in 6"
        :key="n"
        class="animate-pulse rounded-xl border border-gray-200 bg-white"
      >
        <div class="h-44 w-full rounded-t-xl bg-gray-200"></div>
        <div class="space-y-3 p-4">
          <div class="h-4 w-2/3 rounded bg-gray-200"></div>
          <div class="h-3 w-1/2 rounded bg-gray-200"></div>
          <div class="h-3 w-full rounded bg-gray-200"></div>
        </div>
      </div>
    </section>

    <section
      v-if="!loading && filteredSorted.length === 0"
      class="text-gray-600"
    >
      No recipes found.
    </section>

    <section class="make me pretty">
      <article
        v-for="r in filteredSorted"
        :key="r.id"
        class="group overflow-hidden rounded-xl border border-gray-200 bg-white shadow-sm transition hover:shadow-md"
      >
        <div class="h-44 w-full bg-gray-100">
          <img
            :src="r.image"
            :alt="r.title"
            class="h-full w-full object-cover"
          />
        </div>

        <div class="space-y-3 p-4">
          <div class="flex items-start justify-between gap-3">
            <h2 class="line-clamp-2 text-base font-semibold leading-tight">
              {{ r.title }}
            </h2>
            <button
              class="rounded p-1 text-lg text-rose-500 transition hover:scale-105"
              :aria-pressed="isLiked(r.id)"
              :title="isLiked(r.id) ? 'Unlike' : 'Like'"
              @click="toggleLike(r.id)"
            >
              <span v-if="isLiked(r.id)">♥</span>
              <span v-else>♡</span>
            </button>
          </div>

          <div
            class="flex flex-wrap items-center gap-x-3 text-sm text-gray-600"
          >
            <span>{{ r.cuisine }}</span>
            <span class="opacity-50">•</span>
            <span>{{ r.difficulty }}</span>
            <span class="opacity-50">•</span>
            <span>{{ r.servings }} servings</span>
          </div>

          <div class="flex justify-between text-sm">
            <div title="Rating">
              <span class="text-amber-500">{{ stars(r.rating) }}</span>
              <span class="text-gray-500"> ({{ r.reviewCount }})</span>
            </div>
            <div title="Total time">{{ formatMinutes(totalTime(r)) }}</div>
          </div>

          <div class="text-sm text-gray-700">
            <span>
              {{ formatIngredients(r.ingredients) }}
            </span>
          </div>
        </div>
      </article>
    </section>
  </div>
</template>
