<template>
  <div id="app">
    <!-- HEADER -->
    <header class="header">
      <h1 class="logo" @click="goHome" style="cursor: pointer;">
        <i class="fa-solid fa-graduation-cap"></i> LessonHub
      </h1>

      <div class="controls">
        <!-- SEARCH -->
        <input
          v-model="searchTerm"
          class="search"
          placeholder="Search by subject, location, price, or spaces..."
        />

        <!-- SORTING -->
        <div class="sort-group">
          <div class="custom-select">
            <select v-model="sortBy">
              <option value="subject">Subject</option>
              <option value="location">Location</option>
              <option value="price">Price</option>
              <option value="spaces">Spaces</option>
            </select>
            <i class="fa-solid fa-angle-down"></i>
          </div>

          <div class="custom-select">
            <select v-model="sortOrder">
              <option value="asc">Ascending</option>
              <option value="desc">Descending</option>
            </select>
            <i class="fa-solid fa-angle-down"></i>
          </div>
        </div>

        <!-- CART BUTTON -->
        <button
          class="cart-btn"
          @click="showCart = !showCart"
          :disabled="cartItemCount === 0"
        >
          <i class="fa-solid fa-cart-shopping"></i>
          <span>Cart</span>
          <span v-if="cartItemCount" class="cart-badge">
            {{ cartItemCount }}
          </span>
        </button>
      </div>
    </header>

    <!-- MAIN -->
    <main>
      <transition name="fade" mode="out-in">
        <!-- LESSON LIST -->
        <section v-if="!showCart" key="lessons" class="lesson-section">
          <h2>Available Lessons</h2>
          <p class="page-subtitle">
            Browse and book lessons across Hendon, Colindale, Brent Cross and more.
          </p>

          <p class="lesson-count">
            {{ sortedAndFilteredLessons.length }} lesson(s) found
          </p>

          <p
            v-if="!sortedAndFilteredLessons.length && searchTerm"
            class="no-results"
          >
            <i class="fa-solid fa-magnifying-glass"></i>
            No lessons found for "{{ searchTerm }}".
          </p>

          <div v-else class="lesson-grid">
            <div
              class="lesson-card"
              v-for="lesson in sortedAndFilteredLessons"
              :key="lesson.id"
            >
              <img
                :src="lesson.image"
                :alt="lesson.subject + ' lesson icon'"
                class="lesson-img"
              />
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
                  <span
                    v-if="lesson.spaces > 0 && lesson.spaces <= 2"
                    class="badge-low"
                  >
                    Almost full
                  </span>
                </p>
              </div>

              <button
                @click="addToCart(lesson)"
                :disabled="lesson.spaces === 0"
                class="add-btn"
              >
                <i class="fa-solid fa-cart-plus"></i>
                {{ lesson.spaces > 0 ? "Add to Cart" : "Full" }}
              </button>
            </div>
          </div>
        </section>

        <!-- CART + CHECKOUT -->
        <section v-else key="cart" class="cart-section">
          <div class="cart-header">
            <button class="back-btn" @click="showCart = false">
              <i class="fa-solid fa-arrow-left"></i> Back to lessons
            </button>
            <h2>Your Cart</h2>
          </div>

          <p v-if="!cart.length" class="cart-empty">
            Your cart is empty. Add some lessons to continue 🌱
          </p>

          <div v-else>
            <ul class="cart-list">
              <li class="cart-header-row">
                <span>Lesson</span>
                <span>Price</span>
                <span>Quantity</span>
                <span>Line Total</span>
                <span></span>
              </li>

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

                <span class="cart-line-total">
                  £{{ (item.price * item.quantity).toFixed(2) }}
                </span>

                <button
                  class="remove-btn"
                  type="button"
                  @click="removeFromCart(index)"
                >
                  <i class="fa-solid fa-trash"></i>
                </button>
              </li>
            </ul>

            <div class="cart-total-box">
              <h3>
                {{ cartItemCount }} item(s) |
                Total: £{{ cartTotal.toFixed(2) }}
              </h3>
            </div>

            <div class="checkout-box">
              <h3>Checkout</h3>
              <form @submit.prevent="checkout">
                <label>
                  Name
                  <input
                    v-model="name"
                    type="text"
                    placeholder="Name (letters only)"
                    @input="nameError = ''; orderConfirmed = false"
                  />
                </label>
                <p v-if="nameError" class="error">{{ nameError }}</p>

                <label>
                  Phone
                  <input
                    v-model="phone"
                    type="tel"
                    placeholder="Phone (numbers only)"
                    @input="phoneError = ''; orderConfirmed = false"
                  />
                </label>
                <p v-if="phoneError" class="error">{{ phoneError }}</p>

                <button
                  class="checkout-btn"
                  type="submit"
                  :disabled="!cart.length || !isFormValid"
                >
                  <i class="fa-solid fa-paper-plane"></i>
                  Submit Order
                </button>
              </form>

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
export default {
  data() {
    return {
      showCart: false,
      searchTerm: "",
      sortBy: "subject",
      sortOrder: "asc",

      // Main list of 10 lessons used in the application
      lessons: [
        {
          id: 1,
          subject: "Mathematics",
          location: "Hendon",
          price: 100,
          spaces: 5,
          icon: "fa-solid fa-calculator",
          image: "https://img.icons8.com/color/240/calculator--v1.png",
        },
        {
          id: 2,
          subject: "English",
          location: "Colindale",
          price: 80,
          spaces: 5,
          icon: "fa-solid fa-book-open",
          image: "https://img.icons8.com/color/240/book-reading.png",
        },
        {
          id: 3,
          subject: "Biology",
          location: "Golders Green",
          price: 90,
          spaces: 5,
          icon: "fa-solid fa-seedling",
          image: "https://img.icons8.com/color/240/dna-helix.png",
        },
        {
          id: 4,
          subject: "Chemistry",
          location: "Brent Cross",
          price: 70,
          spaces: 5,
          icon: "fa-solid fa-flask",
          image: "https://img.icons8.com/color/240/test-tube.png",
        },
        {
          id: 5,
          subject: "History",
          location: "Hendon",
          price: 50,
          spaces: 5,
          icon: "fa-solid fa-landmark",
          image: "https://img.icons8.com/color/240/scroll.png",
        },
        {
          id: 6,
          subject: "Physics",
          location: "Colindale",
          price: 95,
          spaces: 5,
          icon: "fa-solid fa-atom",
          image: "https://img.icons8.com/color/240/physics.png",
        },
        {
          id: 7,
          subject: "Art",
          location: "Brent Cross",
          price: 60,
          spaces: 5,
          icon: "fa-solid fa-palette",
          image: "https://img.icons8.com/color/240/art-prices.png",
        },
        {
          id: 8,
          subject: "Geography",
          location: "Golders Green",
          price: 85,
          spaces: 5,
          icon: "fa-solid fa-earth-europe",
          image: "https://img.icons8.com/color/240/globe--v1.png",
        },
        {
          id: 9,
          subject: "Computer Science",
          location: "Hendon",
          price: 120,
          spaces: 5,
          icon: "fa-solid fa-code",
          image: "https://img.icons8.com/color/240/source-code.png",
        },
        {
          id: 10,
          subject: "Economics",
          location: "Colindale",
          price: 110,
          spaces: 5,
          icon: "fa-solid fa-chart-line",
          image: "https://img.icons8.com/color/240/economic-improvement.png",
        },
      ],
      cart: [],
      name: "",
      phone: "",
      nameError: "",
      phoneError: "",
      orderConfirmed: false,
    };
  },
  computed: {
  // Apply search term and sorting options to the lessons list
    sortedAndFilteredLessons() {
      const term = this.searchTerm.toLowerCase();

      let filtered = this.lessons.filter((lesson) => {
        if (!term) return true;
        return (
          lesson.subject.toLowerCase().includes(term) ||
          lesson.location.toLowerCase().includes(term) ||
          String(lesson.price).includes(term) ||
          String(lesson.spaces).includes(term)
        );
      });

      const field = this.sortBy;
      const order = this.sortOrder === "asc" ? 1 : -1;

      filtered.sort((a, b) => {
        let x = a[field];
        let y = b[field];

        if (typeof x === "string") x = x.toLowerCase();
        if (typeof y === "string") y = y.toLowerCase();

        if (x < y) return -1 * order;
        if (x > y) return 1 * order;
        return 0;
      });

      return filtered;
    },
    // Total number of lesson units in the cart
    cartItemCount() {
      return this.cart.reduce((sum, item) => sum + item.quantity, 0);
    },
    // Total price of all items in the cart
    cartTotal() {
      return this.cart.reduce(
        (total, item) => total + item.price * item.quantity,
        0
      );
    },
    
    // Used to enable or disable the checkout button
    isFormValid() {
      return this.validateName(this.name) && this.validatePhone(this.phone);
    },
  },
  methods: {
   // Switch back to the lessons view
    goHome() {
      this.showCart = false;
    },
    // Add a lesson to the cart and decrease available spaces
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
      this.showCart = true;
    },
    // Increase quantity for a cart item (if spaces available)
    increaseQuantity(index) {
      const item = this.cart[index];
      const lesson = this.lessons.find((l) => l.id === item.id);
      if (lesson && lesson.spaces > 0) {
        item.quantity += 1;
        lesson.spaces -= 1;
      }
    },
    // Decrease quantity or remove item when it reaches zero
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
    // Remove an item from the cart and restore its spaces
    removeFromCart(index) {
      const item = this.cart[index];
      const lesson = this.lessons.find((l) => l.id === item.id);
      if (lesson) {
        lesson.spaces += item.quantity;
      }
      this.cart.splice(index, 1);
    },
    // Validate that the name contains only letters and spaces
    validateName(value) {
      return /^[A-Za-z\s]+$/.test(value);
    },
    // Validate that the phone number contains only digits
    validatePhone(value) {
      return /^[0-9]+$/.test(value);
    },
    // Handle checkout: validate form, then clear cart and show success message
    checkout() {
      this.nameError = "";
      this.phoneError = "";

      if (!this.validateName(this.name)) {
        this.nameError = "Name must contain letters and spaces only.";
      }
      if (!this.validatePhone(this.phone)) {
        this.phoneError = "Phone must contain numbers only.";
      }

      if (this.nameError || this.phoneError) return;

      this.orderConfirmed = true;
      this.cart = [];
      this.name = "";
      this.phone = "";
    },
  },
};
</script>