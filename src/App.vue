<template>
  <div class="wrapper">
    <div class="droppable" style="margin-right: 30px">
      <button @dragstart="onDragStart($event, buttons[buttons.length - 1])" class="droppable__start-button" draggable="true">
        Button
      </button>
    </div>
    <table>
      <tr v-for="row in rows" :key="`Row-${row}`">
        <td
          v-for="column in columns"
          :key="`Column-${column}`"
          class="droppable"
          @drop="onDrop($event, `${row}.${column}`)"
          @dragover.prevent
          @dragenter.prevent
        >
          <button
            v-for="button in buttons.filter((item) => item.categoryId === `${row}.${column}`)"
            @dblclick="deleteButton(button)"
            @dragstart="onDragStart($event, button)"
            class="droppable__grid-button"
            draggable="true"
            title="Double click to delete"
          >
            Button
          </button>
        </td>
      </tr>
    </table>
  </div>
</template>

<script setup lang="ts">
  import { ref, Ref, onMounted } from "vue";

  interface Item {
    id: number;
    categoryId: string;
  }

  const rows = ref(5);
  const columns = ref(5);
  const buttons: Ref<Item[]> = ref([]);

  function onDragStart(e: DragEvent, item: Item) {
    if (e.dataTransfer) {
      e.dataTransfer.dropEffect = "move";
      e.dataTransfer.effectAllowed = "move";
      e.dataTransfer.setData("itemId", item.id.toString());
    }
  }
  function onDrop(e: DragEvent, categoryId: string) {
    if (e.dataTransfer) {
      const itemId = parseInt(e.dataTransfer.getData("itemId"));
      buttons.value.forEach((i) => {
        if (i.id == itemId) {
          i.categoryId = categoryId;
        }
        return i;
      });
      if (!buttons.value.find((i) => i.categoryId === "0")) {
        buttons.value.push({
          id: buttons.value[buttons.value.length - 1].id + 1,
          categoryId: "0",
        });
      }
    }
  }
  function deleteButton(button: Item) {
    buttons.value = buttons.value.filter((i) => i.id !== button.id);
  }

  onMounted(() => {
    window.addEventListener("beforeunload", () => {
      localStorage.setItem("buttons", JSON.stringify(buttons.value));
    });

    const localButtons = localStorage.getItem("buttons");
    if (localButtons) {
      buttons.value = JSON.parse(localButtons);
    }
    if (!localButtons) {
      buttons.value.push({
        id: 0,
        categoryId: "0",
      });
    }
  });
</script>

<style lang="scss" scoped>
  .wrapper {
    display: flex;
  }
  .droppable {
    background-color: #f3f3f3;
    width: 100px;
    height: 50px;
    &__start-button,
    &__grid-button {
      border-radius: 0px;
      background: #e6b8af;
      width: 100%;
      height: 100%;
    }
    &__grid-button {
      top: 0;
      left: 0;
      position: absolute;
    }
  }

  table {
    border-collapse: collapse;
    td {
      position: relative;
      border: 1px solid black;
    }
  }
</style>
