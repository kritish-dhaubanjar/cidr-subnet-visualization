<template>
  <div>
    <table class="table table-bordered border-primary">
      <tbody>
        <tr>
          <td
            v-for="(digit, index) of digits"
            :key="index"
            class="text-center fw-bold"
            :class="{ disabled: index + 1 + segment > cidr }"
          >
            {{ digit }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { computed } from "@vue/runtime-core";

export default {
  props: {
    mask: Number,
    cidr: Number,
    position: Number,
  },

  setup(props) {
    const digits = computed(() => {
      const binary = props?.mask?.toString(2).padStart(8, 0).split("");
      return binary;
    });

    const segment = computed(() => 8 * props?.position);

    return { digits, segment };
  },
};
</script>
