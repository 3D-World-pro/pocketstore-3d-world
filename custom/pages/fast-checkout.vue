<template>
  <div class="gpay-wrapper">
    <div v-if="loading" class="gpay-status">Loading Google Pay...</div>

    <div ref="buttonContainer" class="gpay-button-container" />

    <div class="gpay-status" v-if="status">{{ status }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted, onBeforeUnmount, watch } from 'vue';

type Env = 'TEST' | 'PRODUCTION';

const props = defineProps({
  amount: { type: String, default: '1.00' },
  currency: { type: String, default: 'USD' },
  merchantName: { type: String, default: 'Example Merchant' },
  environment: { type: String as () => Env, default: 'TEST' },
  // Tokenization (use your gateway in production)
  gateway: { type: String, default: 'example' },
  gatewayMerchantId: { type: String, default: 'BCR2DN5T5CC5PV3C' },
  // Server endpoint to POST the paymentData to (optional)
  processEndpoint: { type: String, default: '/process-payment' },
  // Optional control: autoSendToServer: if true the component will POST paymentData to `processEndpoint`.
  autoSendToServer: { type: Boolean, default: true }
});

const emit = defineEmits<{
  (e: 'success', payload: any): void;
  (e: 'error', error: any): void;
  (e: 'ready', available: boolean): void;
}>();

// Refs / reactive state
const buttonContainer = ref<HTMLElement | null>(null);
const status = ref('');
const loading = ref(true);
let paymentsClient: any = null;
let buttonElement: HTMLElement | null = null;
let scriptEl: HTMLScriptElement | null = null;

// Google Pay configuration
const allowedCardNetworks = ['AMEX', 'DISCOVER', 'MASTERCARD', 'VISA'];
const allowedCardAuthMethods = ['PAN_ONLY', 'CRYPTOGRAM_3DS'];

declare global {
  interface Window { google?: any; }
}
const baseRequest = { apiVersion: 2, apiVersionMinor: 0 };

const tokenizationSpecification = computed(() => ({
  type: 'PAYMENT_GATEWAY',
  parameters: {
    gateway: props.gateway,
    gatewayMerchantId: props.gatewayMerchantId
  }
}));

const cardPaymentMethod = computed(() => ({
  type: 'CARD',
  parameters: {
    allowedAuthMethods: allowedCardAuthMethods,
    allowedCardNetworks: allowedCardNetworks,
    billingAddressRequired: true,
    billingAddressParameters: { format: 'FULL' }
  },
  tokenizationSpecification: tokenizationSpecification.value
}));

function updateStatus(msg: string) {
  status.value = msg;
}

// Dynamically load Google Pay JS if not already present
function loadGooglePayScript(): Promise<void> {
  if (window.google && window.google.payments && window.google.payments.api) {
    return Promise.resolve();
  }
  return new Promise((resolve, reject) => {
    scriptEl = document.createElement('script');
    scriptEl.src = 'https://pay.google.com/gp/p/js/pay.js';
    scriptEl.async = true;
    scriptEl.onload = () => {
      resolve();
    };
    scriptEl.onerror = (ev) => {
      reject(new Error('Failed to load Google Pay script'));
    };
    document.head.appendChild(scriptEl);
  });
}

function initPaymentsClient() {
  paymentsClient = new window.google.payments.api.PaymentsClient({ environment: props.environment });
}

// Check availability and create button if available
async function initGooglePay() {
  try {
    await loadGooglePayScript();
  } catch (err) {
    updateStatus('Could not load Google Pay script.');
    loading.value = false;
    emit('error', err);
    return;
  }

  try {
    initPaymentsClient();
    const isReady = await paymentsClient.isReadyToPay({
      allowedPaymentMethods: ['CARD']
    });

    if (isReady.result) {
      emit('ready', true);
      createAndAddButton();
      updateStatus('Google Pay is available. Click the button to pay.');
    } else {
      emit('ready', false);
      updateStatus('Google Pay is not available in this environment/profile.');
    }
  } catch (err) {
    updateStatus('Error checking Google Pay availability.');
    emit('error', err);
    console.error(err);
  } finally {
    loading.value = false;
  }
}

function createAndAddButton() {
  if (!paymentsClient) return;
  // remove existing button if any
  if (buttonElement && buttonContainer.value?.contains(buttonElement)) {
    buttonContainer.value.removeChild(buttonElement);
    buttonElement = null;
  }

  // createButton supports an onClick handler
  buttonElement = paymentsClient.createButton({
    onClick: onGooglePayButtonClicked,
    buttonColor: 'default',
    buttonType: 'long'
  });
  if (buttonContainer.value && buttonElement) {
    buttonContainer.value.appendChild(buttonElement);
  }
}

function getPaymentDataRequest(totalPrice = props.amount, currencyCode = props.currency) {
  const paymentDataRequest: any = Object.assign({}, baseRequest);
  paymentDataRequest.allowedPaymentMethods = [cardPaymentMethod.value];
  paymentDataRequest.transactionInfo = {
    totalPriceStatus: 'FINAL',
    totalPrice,
    currencyCode
  };
  paymentDataRequest.merchantInfo = {
    merchantName: props.merchantName
    // merchantId: '...' // set in production if you have one
  };
  return paymentDataRequest;
}

function onGooglePayButtonClicked() {
  if (!paymentsClient) {
    updateStatus('Payments client not initialized.');
    return;
  }
  const request = getPaymentDataRequest();
  paymentsClient.loadPaymentData(request)
      .then((paymentData: any) => {
        handlePaymentSuccess(paymentData);
      })
      .catch((err: any) => {
        console.error('loadPaymentData failed', err);
        updateStatus('Payment cancelled or failed; check console.');
        emit('error', err);
      });
}

async function handlePaymentSuccess(paymentData: any) {
  // token available at paymentData.paymentMethodData.tokenizationData.token
  console.log('Google Pay paymentData', paymentData);
  updateStatus('Payment authorized. Processing...');

  emit('success', paymentData);

  if (props.autoSendToServer) {
    try {
      const res = await fetch(props.processEndpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ paymentData })
      });
      const body = await res.json().catch(() => null);
      console.log('Server response:', body);
      updateStatus('Server processed payment.');
    } catch (err) {
      console.error('Error sending token to server', err);
      updateStatus('Error sending token to server; see console.');
      emit('error', err);
    }
  }
}

onMounted(() => {
  initGooglePay();
});

onBeforeUnmount(() => {
  // remove injected script if we added one
  if (scriptEl && scriptEl.parentNode) {
    scriptEl.parentNode.removeChild(scriptEl);
    scriptEl = null;
  }
  // clear payments client and DOM button
  paymentsClient = null;
  if (buttonElement && buttonContainer.value?.contains(buttonElement)) {
    buttonContainer.value.removeChild(buttonElement);
    buttonElement = null;
  }
});

</script>

<style scoped>
.gpay-wrapper {
  max-width: 560px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}
.gpay-button-container {
  margin-top: 12px;
}
.gpay-status {
  margin-top: 12px;
  color: #333;
}
</style>