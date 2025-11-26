<template>
  <div id="app">
    <!-- ========================= HEADER ========================= -->
    <header class="header">
      <!-- Logo also acts as a "home" button to return to lesson list -->
      <h1 class="logo" @click="goHome" style="cursor: pointer;">
        <i class="fa-solid fa-graduation-cap"></i> LessonHub
      </h1>

      <div class="controls">
        <!-- SEARCH -->
        <input
          v-model="searchTerm"
          class="search"
          placeholder="Search by subject, location, price, or spaces..."
          aria-label="Search lessons"
        />

        <!-- SORTING CONTROLS -->
        <div class="sort-group">
          <!-- Sort by field: subject, location, price or spaces -->
          <div class="custom-select">
            <select v-model="sortBy">
              <option value="subject">Subject</option>
              <option value="location">Location</option>
              <option value="price">Price</option>
              <option value="spaces">Spaces</option>
            </select>
            <i class="fa-solid fa-angle-down"></i>
          </div>

          <!-- Sort order: ascending or descending -->
          <div class="custom-select">
            <select v-model="sortOrder">
              <option value="asc">Ascending</option>
              <option value="desc">Descending</option>
            </select>
            <i class="fa-solid fa-angle-down"></i>
          </div>
        </div>

        <!-- CART BUTTON (disabled when cart is empty) -->
        <button
          class="cart-btn"
          @click="toggleCart"
          :disabled="cartItemCount === 0"
        >
          <span class="cart-icon">
            <i class="fa-solid fa-cart-shopping"></i>
            <span v-if="cartItemCount" class="cart-badge">
              {{ cartItemCount }}
            </span>
          </span>
          <span>Cart</span>
        </button>
      </div>
    </header>

    <!-- ========================= MAIN CONTENT ========================= -->
    <main>
      <!-- Little feedback line when adding items -->
      <p v-if="feedbackMessage" class="feedback-message">
        {{ feedbackMessage }}
      </p>

      <!-- Fade transition between lessons and cart views -->
      <transition name="fade" mode="out-in">
        <!-- ===================== LESSON LIST VIEW ===================== -->
        <section v-if="!showCart" key="lessons" class="lesson-section">
          <h2>Available Lessons</h2>
          <p class="page-subtitle">
            Browse and book lessons across Hendon, Colindale, Brent Cross and more.
          </p>

          <!-- Show how many lessons match the current search/filter -->
          <p class="lesson-count">
            {{ sortedAndFilteredLessons.length }} lesson(s) found
          </p>

          <!-- Friendly message when no lessons match the search term -->
          <p
            v-if="!sortedAndFilteredLessons.length && searchTerm"
            class="no-results"
          >
            <i class="fa-solid fa-magnifying-glass"></i>
            No lessons found for "{{ searchTerm }}".
          </p>

          <!-- Lesson cards -->
          <div v-else class="lesson-grid">
            <div
              class="lesson-card"
              v-for="lesson in sortedAndFilteredLessons"
              :key="lesson.id"
            >
              <!-- Lesson icon/image -->
              <img
                :src="lesson.image"
                :alt="lesson.subject + ' lesson icon'"
                class="lesson-img"
              />

              <!-- Lesson information -->
              <div class="lesson-info">
                <h3>
                  <i :class="lesson.icon"></i>
                  {{ lesson.subject }}
                </h3>

                <p>
                  <i class="fa-solid fa-location-dot"></i>
                  {{ lesson.location }}
                </p>

                <p>£{{ lesson.price }}</p>

                <p>
                  <i class="fa-solid fa-users"></i>
                  Spaces: {{ lesson.spaces }}
                  <!-- “Almost full” badge when only 1–2 spaces remain -->
                  <span
                    v-if="lesson.spaces > 0 && lesson.spaces <= 2"
                    class="badge-low"
                  >
                    Almost full
                  </span>
                </p>
              </div>

              <!-- Add to Cart button -->
              <button
                class="add-btn"
                @click="addToCart(lesson)"
                :disabled="lesson.spaces === 0"
              >
                <i class="fa-solid fa-cart-plus"></i>
                {{ lesson.spaces > 0 ? "Add to Cart" : "Full" }}
              </button>
            </div>
          </div>
        </section>

        <!-- ===================== CART + CHECKOUT VIEW ===================== -->
        <section v-else key="cart" class="cart-section">
          <div class="cart-header">
            <!-- Navigate back to lesson list -->
            <button class="back-btn" @click="showCart = false">
              <i class="fa-solid fa-arrow-left"></i> Back to lessons
            </button>
            <h2>Your Cart</h2>
          </div>

          <!-- Empty cart message -->
          <p v-if="!cart.length" class="cart-empty">
            Your cart is empty. Add some lessons to continue 🌱
          </p>

          <div v-else>
            <!-- Cart items “table” -->
            <ul class="cart-list">
              <li class="cart-header-row">
                <span>Lesson</span>
                <span>Price</span>
                <span>Quantity</span>
                <span>Line Total</span>
                <span></span>
              </li>

              <!-- One row per cart item -->
              <li
                v-for="(item, index) in cart"
                :key="item.id"
                class="cart-item"
              >
                <div class="cart-item-info">
                  <img
                    :src="item.image"
                    :alt="item.subject + ' lesson icon'"
                    class="cart-thumb"
                  />
                  <div>
                    <h4>{{ item.subject }}</h4>
                    <p>
                      <i class="fa-solid fa-location-dot"></i>
                      {{ item.location }}
                    </p>
                  </div>
                </div>

                <span class="cart-price">£{{ item.price }}</span>

                <!-- Quantity controls -->
                <div class="cart-qty">
                  <button
                    class="qty-btn"
                    type="button"
                    @click="decreaseQuantity(index)"
                  >
                    -
                  </button>
                  <span class="qty-value">{{ item.quantity }}</span>
                  <button
                    class="qty-btn"
                    type="button"
                    @click="increaseQuantity(index)"
                  >
                    +
                  </button>
                </div>

                <!-- Line total = price * quantity -->
                <span class="cart-line-total">
                  £{{ (item.price * item.quantity).toFixed(2) }}
                </span>

                <!-- Remove the item completely from the cart -->
                <button
                  class="remove-btn"
                  type="button"
                  @click="removeFromCart(index)"
                >
                  <i class="fa-solid fa-trash"></i>
                </button>
              </li>
            </ul>

            <!-- Overall cart summary -->
            <div class="cart-total-box">
              <h3>
                {{ cartItemCount }} item(s) |
                Total: £{{ cartTotal.toFixed(2) }}
              </h3>
            </div>

            <!-- ===================== CHECKOUT FORM ===================== -->
            <div class="checkout-box">
              <h3>Checkout</h3>

              <form @submit.prevent="checkout">
                <!-- Name input -->
                <label>
                  Name
                  <input
                    v-model="name"
                    type="text"
                    placeholder="Name (letters only)"
                    @input="clearNameError"
                  />
                </label>
                <p v-if="nameError" class="error">{{ nameError }}</p>

                <!-- Phone input -->
                <label>
                  Phone
                  <input
                    v-model="phone"
                    type="tel"
                    placeholder="Phone (numbers only)"
                    @input="clearPhoneError"
                  />
                </label>
                <p v-if="phoneError" class="error">{{ phoneError }}</p>

                <!-- Checkout button -->
                <button
                  class="checkout-btn"
                  type="submit"
                  :disabled="!cart.length || !isFormValid"
                >
                  <i class="fa-solid fa-paper-plane"></i>
                  Submit Order
                </button>
              </form>

              <!-- Success message after a valid checkout -->
              <p v-if="orderConfirmed" class="success">
                Order Confirmed Successfully!
              </p>
            </div>
          </div>
        </section>
      </transition>
    </main>
  </div>
</template>

<script>
const API_BASE_URL = "https://YOUR-URL.onrender.com";

export default {
  name: "App",

  data() {
    return {
      showCart: false,

      searchTerm: "",
      sortBy: "subject",
      sortOrder: "asc",

      feedbackMessage: "",
      lessons: [],

      cart: [],
      name: "",
      phone: "",
      nameError: "",
      phoneError: "",
      orderConfirmed: false,
    };
  },

  computed: {
    // Lessons are filtered by backend search, we only sort here
    sortedAndFilteredLessons() {
      const field = this.sortBy;
      const order = this.sortOrder === "asc" ? 1 : -1;
      const copy = [...this.lessons];

      copy.sort((a, b) => {
        let x = a[field];
        let y = b[field];

        if (typeof x === "string") x = x.toLowerCase();
        if (typeof y === "string") y = y.toLowerCase();

        if (x < y) return -1 * order;
        if (x > y) return 1 * order;
        return 0;
      });

      return copy;
    },

    cartItemCount() {
      return this.cart.reduce((sum, item) => sum + item.quantity, 0);
    },

    cartTotal() {
      return this.cart.reduce(
        (total, item) => total + item.price * item.quantity,
        0
      );
    },

    isFormValid() {
      return (
        this.name.length > 0 &&
        this.phone.length > 0 &&
        this.validateName(this.name) &&
        this.validatePhone(this.phone)
      );
    },
  },

  watch: {
    // search-as-you-type → call backend /search
    searchTerm() {
      this.fetchLessons();
    },
  },

  created() {
    this.fetchLessons();
  },

  methods: {
    // =============== API CALLS ===============

    async fetchLessons() {
      try {
        const term = this.searchTerm.trim();
        const endpoint = term
          ? `${API_BASE_URL}/search?q=${encodeURIComponent(term)}`
          : `${API_BASE_URL}/lessons`;

        const response = await fetch(endpoint);
        if (!response.ok) throw new Error("Failed to load lessons");

        const data = await response.json();
        this.lessons = data;
      } catch (err) {
        console.error("fetchLessons error:", err);
      }
    },

    // =============== UI HELPERS ===============

    goHome() {
      this.showCart = false;
    },

    toggleCart() {
      if (this.cartItemCount > 0) {
        this.showCart = !this.showCart;
      }
    },

    addToCart(lesson) {
      if (lesson.spaces <= 0) return;

      const existing = this.cart.find((item) => item.id === lesson.id);

      if (existing) {
        existing.quantity += 1;
      } else {
        this.cart.push({
          id: lesson.id,
          subject: lesson.subject,
          location: lesson.location,
          price: lesson.price,
          image: lesson.image,
          quantity: 1,
        });
      }

      lesson.spaces -= 1;

      this.feedbackMessage = `${lesson.subject} added to cart`;
      setTimeout(() => {
        this.feedbackMessage = "";
      }, 2000);
    },

    increaseQuantity(index) {
      const item = this.cart[index];
      const lesson = this.lessons.find((l) => l.id === item.id);

      if (lesson && lesson.spaces > 0) {
        item.quantity += 1;
        lesson.spaces -= 1;
      }
    },

    decreaseQuantity(index) {
      const item = this.cart[index];
      const lesson = this.lessons.find((l) => l.id === item.id);

      if (item.quantity > 1) {
        item.quantity -= 1;
        if (lesson) lesson.spaces += 1;
      } else {
        this.removeFromCart(index);
      }
    },

    removeFromCart(index) {
      const item = this.cart[index];
      const lesson = this.lessons.find((l) => l.id === item.id);

      if (lesson) {
        lesson.spaces += item.quantity;
      }

      this.cart.splice(index, 1);
    },

    validateName(value) {
      return /^[A-Za-z\s]+$/.test(value);
    },

    validatePhone(value) {
      return /^[0-9]+$/.test(value);
    },

    clearNameError() {
      this.nameError = "";
      this.orderConfirmed = false;
    },

    clearPhoneError() {
      this.phoneError = "";
      this.orderConfirmed = false;
    },

    // =============== CHECKOUT (POST + PUT) ===============

    async checkout() {
      this.nameError = "";
      this.phoneError = "";
      this.orderConfirmed = false;

      if (!this.validateName(this.name)) {
        this.nameError = "Name must contain letters and spaces only.";
      }

      if (!this.validatePhone(this.phone)) {
        this.phoneError = "Phone must contain numbers only.";
      }

      if (this.nameError || this.phoneError) return;

      const orderPayload = {
        name: this.name,
        phone: this.phone,
        items: this.cart.map((item) => ({
          id: item.id,
          quantity: item.quantity,
        })),
        total: this.cartTotal,
      };

      try {
        // 1) save order
        const res = await fetch(`${API_BASE_URL}/orders`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(orderPayload),
        });

        if (!res.ok) throw new Error("Failed to submit order");

        // 2) update spaces in DB for each lesson
        await Promise.all(
          this.cart.map(async (item) => {
            const lesson = this.lessons.find((l) => l.id === item.id);
            if (!lesson) return;

            await fetch(`${API_BASE_URL}/lessons/${lesson.id}`, {
              method: "PUT",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify({ spaces: lesson.spaces }),
            });
          })
        );

        this.orderConfirmed = true;
        this.cart = [];
        this.name = "";
        this.phone = "";

        // refresh from backend so spaces match DB
        this.fetchLessons();
      } catch (err) {
        console.error("Checkout error:", err);
        alert("There was a problem submitting your order. Please try again.");
      }
    },
  },
};
</script>