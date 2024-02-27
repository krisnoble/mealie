<template>
  <div>
    <div class="d-flex justify-end flex-wrap align-stretch">
      <v-card v-if="!landscape" width="50%" flat class="d-flex flex-column justify-center align-left">
        <v-card-text>
          <v-card-title class="headline pa-0">
            {{ recipe.name }}
            <!-- <RecipeRating :key="recipe.slug" v-model="recipe.rating" :name="recipe.name" :slug="recipe.slug" /> -->
          </v-card-title>
          <SafeMarkdown class="my-8" :source="recipe.description" />
          <div class="d-flex justify-left mt-5">
            <RecipeTimeCard
              class="d-flex justify-left flex-wrap"
              :class="true ? undefined : 'force-bottom'"
              :prep-time="recipe.prepTime"
              :total-time="recipe.totalTime"
              :perform-time="recipe.performTime"
              color="grey-lighten-1 custom-transparent"
            />
          </div>
          <v-btn
            v-if="recipe.orgURL && !isCookMode"
            dense
            small
            flat
            dark
            :hover="false"
            type="label"
            :ripple="false"
            elevation="0"
            :href="recipe.orgURL"
            :color="$vuetify.theme.dark ? '#007575' : '#0aa'"
            target="_blank"
            class="rounded mt-4"
          >
            {{ $t("recipe.original-url") }}
          </v-btn>
        </v-card-text>
      </v-card>
      <v-img
        :key="imageKey"
        :max-width="landscape ? null : '50%'"
        min-height="50"
        :height="hideImage ? undefined : imageHeight"
        :src="recipeImageUrl"
        class="d-print-none"
        @error="hideImage = true"
      >
      </v-img>
    </div>
    <v-divider></v-divider>
    <RecipeActionMenu
      :recipe="recipe"
      :slug="recipe.slug"
      :recipe-scale="recipeScale"
      :locked="isOwnGroup && user.id !== recipe.userId && recipe.settings.locked"
      :name="recipe.name"
      :logged-in="isOwnGroup"
      :open="isEditMode"
      :recipe-id="recipe.id"
      class="ml-auto mt-n8 pb-4"
      @close="setMode(PageMode.VIEW)"
      @json="toggleEditMode()"
      @edit="setMode(PageMode.EDIT)"
      @save="$emit('save')"
      @delete="$emit('delete')"
      @print="printRecipe"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, useContext, computed, ref, watch } from "@nuxtjs/composition-api";
import { useLoggedInState } from "~/composables/use-logged-in-state";
import RecipeRating from "~/components/Domain/Recipe/RecipeRating.vue";
import RecipeLastMade from "~/components/Domain/Recipe/RecipeLastMade.vue";
import RecipeActionMenu from "~/components/Domain/Recipe/RecipeActionMenu.vue";
import RecipeTimeCard from "~/components/Domain/Recipe/RecipeTimeCard.vue";
import { useStaticRoutes } from "~/composables/api";
import { Recipe } from "~/lib/api/types/recipe";
import { NoUndefinedField } from "~/lib/api/types/non-generated";
import { usePageState, usePageUser, PageMode, EditorMode } from "~/composables/recipe-page/shared-state";
export default defineComponent({
  components: {
    RecipeTimeCard,
    RecipeActionMenu,
    RecipeRating,
    RecipeLastMade,
  },
  props: {
    recipe: {
      type: Object as () => NoUndefinedField<Recipe>,
      required: true,
    },
    recipeScale: {
      type: Number,
      default: 1,
    },
    landscape: {
      type: Boolean,
      default: false,
    },
  },
  setup(props) {
    const { $vuetify } = useContext();
    const { recipeImage } = useStaticRoutes();
    const { imageKey, pageMode, editMode, setMode, toggleEditMode, isEditMode } = usePageState(props.recipe.slug);
    const { user } = usePageUser();
    const { isOwnGroup } = useLoggedInState();

    function printRecipe() {
      window.print();
    }

    const hideImage = ref(false);
    const imageHeight = computed(() => {
      return $vuetify.breakpoint.xs ? "200" : "400";
    });

    const recipeImageUrl = computed(() => {
      return recipeImage(props.recipe.id, props.recipe.image, imageKey.value);
    });

    watch(
      () => recipeImageUrl.value,
      () => {
        hideImage.value = false;
      }
    );

    return {
      isOwnGroup,
      setMode,
      toggleEditMode,
      recipeImage,
      imageKey,
      user,
      PageMode,
      pageMode,
      EditorMode,
      editMode,
      printRecipe,
      imageHeight,
      hideImage,
      isEditMode,
      recipeImageUrl,
    };
  },
});
</script>
