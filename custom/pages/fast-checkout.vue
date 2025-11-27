<template>
  <div>
    <div id="payment-request-button"></div>
    <p v-if="!googlePayAvailable">Google Pay ist nicht verfügbar.</p>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { loadStripe } from '@stripe/stripe-js'
import { useLocalStorage } from '@vueuse/core'

// Stripe Public Key
const stripePublicKey = 'pk_test_51QJlDLR5d3xw1mbRyaq24na2qlIiPpPjYfnq9m6TL2PSmhjvt2DFh0DHK6aUGaw5M4Mp81wlCOhpqW2jcvZfi3YQ000eWIgGiJ'

// Google Pay verfügbar?
const googlePayAvailable = ref(false)

// Warenkorb aus LocalStorage
const cart = useLocalStorage('cart', [])

// Gesamtbetrag in Cent berechnen
const totalAmount = computed(() => {
  return cart.value.reduce((sum, item) => {
    // Preis pro Produkt * Menge, in Cent
    return sum + Math.round(item.product.price * 100) * (item.qty || 1)
  }, 0)
})

// Gesamtbezeichnung für Button
const totalLabel = computed(() => {
  if (!cart.value.length) return 'Warenkorb'
  const item = cart.value[0]
  return `${item.product.name}${cart.value.length > 1 ? ' + weitere' : ''}`
})

onMounted(async () => {
  const stripe = await loadStripe(stripePublicKey)

  const paymentRequest = stripe.paymentRequest({
    country: 'DE',
    currency: 'eur',
    total: {
      label: totalLabel.value,
      amount: totalAmount.value,
    },
    requestPayerName: true,
    requestPayerEmail: true,
  })

  const canPay = await paymentRequest.canMakePayment()
  if (canPay) {
    googlePayAvailable.value = true
    const elements = stripe.elements()
    const prButton = elements.create('paymentRequestButton', { paymentRequest })
    prButton.mount('#payment-request-button')
  } else {
    googlePayAvailable.value = false
  }

  paymentRequest.on('paymentmethod', async (ev) => {
    try {
      // Hier würdest du ev.paymentMethod.id an dein Backend senden
      // z.B. PaymentIntent erstellen mit der gesamten Cart
      console.log('PaymentMethod:', ev.paymentMethod.id)
      console.log('Cart:', cart.value)
      ev.complete('success')
    } catch (error) {
      ev.complete('fail')
      console.error(error)
    }
  })
})
</script>
