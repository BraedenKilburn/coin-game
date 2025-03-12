<script setup lang="ts">
import { ref, computed, onMounted, type Component } from 'vue'
import { penny, nickel, dime, quarter } from '@/components/icons/coins'

interface Coin {
  name: string
  value: number
  component: Component
}

// Define the coins
const coins: Coin[] = [
  {
    name: 'Penny',
    value: 0.01,
    component: penny,
  },
  {
    name: 'Nickel',
    value: 0.05,
    component: nickel,
  },
  {
    name: 'Dime',
    value: 0.1,
    component: dime,
  },
  {
    name: 'Quarter',
    value: 0.25,
    component: quarter,
  },
]

// Game state
const selectedCoins = ref<Record<number, number>>({
  0.01: 0,
  0.05: 0,
  0.1: 0,
  0.25: 0,
})

/**
 * Calculate the current amount of coins selected
 * @returns {number} The current value of coins selected
 */
const currentAmount = computed(() => {
  return Object.entries(selectedCoins.value).reduce(
    (total, [value, count]) => total + parseFloat(value) * count,
    0,
  )
})

/**
 * Format the amount to 2 decimal places
 * @param {number} amount - The amount to format
 * @returns {string} The formatted amount
 */
const formatAmount = (amount: number): string => {
  return amount.toFixed(2)
}

/**
 * Generate a random amount between 0.01 and 0.99
 * @returns {number} The random amount
 */
const generateRandomAmount = (): number => {
  // Generate random cents between 1 and 99
  const cents = Math.floor(Math.random() * 99) + 1
  return cents / 100
}

/**
 * The message to display to the user
 */
const message = ref('')

/**
 * The type of message to display
 */
const messageType = ref('')

/**
 * Whether the game has been won
 */
const isGameWon = ref(false)

/**
 * Select a coin
 * @param {Coin} coin - The coin to select
 */
const selectCoin = (coin: Coin): void => {
  if (isGameWon.value) return

  selectedCoins.value[coin.value]++
  message.value = ''
}

/**
 * Remove a coin
 * @param {Coin} coin - The coin to remove
 * @param {Event} event - The event object
 */
const removeCoin = (coin: Coin, event?: Event): void => {
  if (event) event.preventDefault() // Prevent context menu on right-click
  if (isGameWon.value || selectedCoins.value[coin.value] <= 0) return

  selectedCoins.value[coin.value]--
  message.value = ''
}

/**
 * The target amount to reach
 */
const targetAmount = ref(0)

/**
 * Check the current total against the target amount, and set the message and game state accordingly
 */
function checkAnswer() {
  const difference = Math.abs(currentAmount.value - targetAmount.value)

  // Using a small epsilon to account for floating point comparison issues
  if (difference < 0.001) {
    message.value = 'Correct! You made the exact change!'
    messageType.value = 'success'
    isGameWon.value = true
  } else if (currentAmount.value < targetAmount.value) {
    message.value = 'Not enough. Add more coins!'
    messageType.value = 'error'
  } else {
    message.value = 'Too much! Remove some coins!'
    messageType.value = 'error'
  }
}

/**
 * Clear the current selection of coins
 */
function clearSelection() {
  Object.keys(selectedCoins.value).forEach((key) => {
    selectedCoins.value[parseFloat(key)] = 0
  })
  message.value = ''
}

/**
 * Generate a new target amount and clear the selection
 */
function reset() {
  targetAmount.value = generateRandomAmount()
  clearSelection()
  isGameWon.value = false
}

// Initialize the game
onMounted(() => {
  reset()
})
</script>

<template>
  <div class="coin-game">
    <h1>Make Exact Change</h1>

    <div class="target-amount">
      <h2>Target: ${{ formatAmount(targetAmount) }}</h2>
      <p>Your selection: ${{ formatAmount(currentAmount) }}</p>
      <p v-if="message" :class="['status-message', messageType]">{{ message }}</p>
    </div>

    <div class="coins-container">
      <div
        v-for="coin in coins"
        :key="coin.value"
        class="coin"
        @click="selectCoin(coin)"
        @contextmenu="removeCoin(coin, $event)"
        :class="{ disabled: isGameWon }"
      >
        <div class="coin-inner">
          <component :is="coin.component" :alt="coin.name" />
          <div class="coin-details">
            <p>{{ coin.name }}</p>
            <p>${{ formatAmount(coin.value) }}</p>
          </div>
        </div>
        <div class="coin-controls">
          <button
            v-if="selectedCoins[coin.value] > 0"
            class="remove-coin"
            @click.stop="removeCoin(coin)"
            :disabled="isGameWon || selectedCoins[coin.value] <= 0"
          >
            -
          </button>
          <span class="coin-count" v-if="selectedCoins[coin.value] > 0">
            {{ selectedCoins[coin.value] }}
          </span>
        </div>
      </div>
    </div>

    <div class="controls">
      <button class="secondary" @click="reset">New Game</button>
      <button class="destructive" @click="clearSelection" :disabled="isGameWon">Clear</button>
      <button class="primary" @click="checkAnswer" :disabled="isGameWon">Check</button>
    </div>
  </div>
</template>

<style scoped lang="scss">
.coin-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;

  > * {
    text-align: center;
  }

  h1 {
    color: var(--color-heading);
  }

  .target-amount {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.75rem;
    color: var(--color-text);

    h2 {
      color: var(--color-heading);
    }

    .status-message {
      font-weight: bold;

      &.success {
        color: var(--color-success);
      }

      &.error {
        color: var(--color-error);
      }
    }
  }

  .coins-container {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 2.5rem;
    margin-bottom: 2.5rem;

    .coin {
      position: relative;
      transition: transform 0.2s ease;
      cursor: pointer;

      &:hover {
        transform: scale(1.05);
      }

      &.disabled {
        opacity: 0.7;
        cursor: not-allowed;
      }

      .coin-inner {
        display: flex;
        flex-direction: column;
        align-items: center;
        height: 100%;

        svg {
          width: 200px;
          height: 200px;
          padding: 10px;
        }

        .coin-details {
          display: flex;
          flex-direction: column;
          align-items: center;
          gap: 0.25rem;

          p {
            font-size: 1.2rem;
            color: var(--color-text);
          }
        }
      }

      .coin-controls {
        position: absolute;
        top: 0;
        right: 0;
        transform: translate(50%, -50%);
        display: flex;
        align-items: center;
        gap: 0.5rem;
        padding: 0.5rem;

        .remove-coin {
          width: 30px;
          height: 30px;
          border-radius: 50%;
          background-color: var(--color-error);
          color: white;
          font-size: 1.2rem;
          padding: 0;
          display: flex;
          align-items: center;
          justify-content: center;
          cursor: pointer;
          border: none;
          transition: all 0.2s ease;

          &:hover:not(:disabled) {
            transform: scale(1.1);
            background-color: var(--color-error-dark);
          }

          &:disabled {
            opacity: 0.5;
            cursor: not-allowed;
          }
        }

        .coin-count {
          background-color: var(--color-background-soft);
          color: var(--color-heading);
          border-radius: 50%;
          width: 30px;
          height: 30px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 1rem;
          font-weight: bold;
        }
      }
    }
  }

  .controls {
    display: flex;
    justify-content: center;
    gap: 26px;

    button {
      padding: 10px 20px;
      border: none;
      border-radius: 4px;
      background-color: var(--color-background-soft);
      color: white;
      cursor: pointer;
      font-size: 1.75rem;
      transition: background-color 0.2s ease;

      &:hover {
        background-color: var(--color-background-mute);
      }

      &:disabled {
        opacity: 0.3;
        cursor: not-allowed;
      }

      &.primary {
        background-color: var(--color-success);
      }

      &.destructive {
        background-color: var(--color-error);
      }

      &.secondary {
        background-color: var(--color-background-soft);
      }
    }
  }
}
</style>
