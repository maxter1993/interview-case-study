<template>
  <breeze-authenticated-layout>
    <template #header>
      <h2 class="font-semibold text-xl text-gray-800 leading-tight">Cart</h2>
    </template>

    <div class="py-12">
      <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
        <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg p-4">
          <table class="table-auto w-full">
            <thead>
              <tr class="border-b">
                <th class="p-2">Product</th>
                <th class="p-2">Quantity</th>
                <th class="p-2">Price</th>
                <th class="p-2">Total</th>
                <th class="p-2"></th>
              </tr>
            </thead>

            <tbody v-if="cart && items.length > 0">
              <tr v-for="item in items" :key="item.id">
                <td class="p-2 text-center">{{ item.product.name }}</td>
                <td class="p-2 text-center">{{ item.quantity }}</td>
                <td class="p-2 text-center">{{ item.price }}</td>
                <td class="p-2 text-center">
                  {{ parseFloat(item.quantity * item.price).toFixed(2) }}
                </td>
                <td class="p-2 text-center">
                  <button
                    type="button"
                    class="w-max p-2 mx-2 sm:rounded-lg shadow-sm bg-red-400 text-white"
                    @click="removeItem(item.id)"
                  >
                    Remove
                  </button>
                </td>
              </tr>
            </tbody>

            <tbody v-else>
              <tr>
                <td class="text-center p-2" colspan="5">No items in cart.</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <div class="pb-12">
      <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
        <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg p-4">
          <div class="flex flex-row content-around items-stretch">
            <div class="flex-grow self-center">
              <h3 class="text-2xl font-bold">Total&nbsp;{{ total }}</h3>
            </div>
            <div class="self-center">
              <button
                type="button"
                class="bg-white p-4 mx-2 sm:rounded-lg shadow-sm bg-indigo-400 text-white"
                @click="placeOrder(cart)"
              >
                Place Order
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </breeze-authenticated-layout>
</template>

<script>
import BreezeAuthenticatedLayout from "@/Layouts/Authenticated";

export default {
  components: {
    BreezeAuthenticatedLayout,
  },

  data() {
    return {
      items: [],
      total: 0,
    };
  },

  computed: {
    getCartTotal() {
      return parseFloat(this.total).toFixed(2);
    },
  },

  props: {
    cart: Object,
  },

  methods: {
    placeOrder(cart) {
      if (this.items.length == 0) {
        this.$swal("Hey!", "There is no item in the cart!", "warning");
      } else {
        axios
          .post(route("order.place"), {
            cart_id: cart.id,
          })
          .then((res) => {
            if (res.status === 200) {
              this.clearCart();
              this.$swal("Done!", "Placed Order!", "success").then((result) => {
                if (result.isConfirmed) {
                  console.log(result);
                  window.location.href = route("order.show", {
                    id: res.data.id,
                  });
                }
              });
            }
          })
          .catch((err) => {
            this.$swal("Oops!", err.message, "error");
          });
      }
    },

    removeItem(cart_item_id) {
      axios
        .delete(route("cart.item.delete"), {
          params: {
            cart_item_id: cart_item_id,
          },
        })
        .then((res) => {
          if (res.status === 200) {
            this.items = this.items.filter((item) => {
              return item.id != cart_item_id;
            });
            this.calculateTotal();
            this.$swal("Done!", "Removed from cart.", "success");
          }
        })
        .catch((err) => {
          this.$swal("Oops!", err.message, "error");
        });
    },

    calculateTotal() {
      var total = 0;
      this.items.forEach((item) => {
        total += item.quantity * item.price;
      });
      this.total = total;
    },

    clearCart() {
      this.total = 0;
      this.items = [];
    },
  },

  mounted() {
    this.items = this.cart && this.cart.items ? this.cart.items : [];
    this.total = this.cart && this.cart.total ? this.cart.total : 0;
  },
};
</script>
