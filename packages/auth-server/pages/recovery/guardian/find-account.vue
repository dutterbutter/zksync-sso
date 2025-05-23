<template>
  <div class="min-h-screen">
    <header class="max-w-[1920px] mx-auto mb-12">
      <app-generic-nav />
    </header>
    <main class="max-w-[900px] mx-auto flex flex-col gap-6">
      <CommonStepper
        :current-step="currentStep"
        :total-steps="6"
        :disabled-steps="disabledSteps"
      />

      <div class="flex flex-col items-center gap-8 mt-4">
        <h1 class="text-3xl font-medium text-neutral-900 dark:text-neutral-100">
          {{ stepTitle }}
        </h1>

        <!-- Step 1: Input Guardian Address -->
        <div
          v-if="currentStep === 1"
          class="w-full max-w-md flex flex-col gap-6"
        >
          <div class="flex flex-col gap-2">
            <label
              for="guardianAddress"
              class="text-sm text-neutral-700 dark:text-neutral-300"
            >
              Insert your guardian address
            </label>
            <ZkInput
              id="guardianAddress"
              v-model="guardianAddress"
              placeholder="0x..."
              :error="!!guardianAddressError"
              :messages="guardianAddressError ? [guardianAddressError] : undefined"
              @input="validateGuardianAddress"
            />
          </div>

          <ZkLink
            class="text-sm text-center w-fit text-neutral-700 dark:text-neutral-300 hover:text-neutral-900 dark:hover:text-neutral-100 transition-colors hover:border-b-neutral-900 dark:hover:border-b-neutral-100"
            href="/recovery/guardian/unknown-account"
          >
            I don't remember my address or the guardian address
          </ZkLink>

          <ZkButton
            class="w-full"
            :disabled="!isValidGuardianAddress"
            :loading="getGuardedAccountsInProgress"
            @click="handleGuardianContinue"
          >
            Continue
          </ZkButton>
        </div>

        <!-- Step 2: Choose Account Address -->
        <div
          v-if="currentStep === 2"
          class="w-full max-w-md flex flex-col gap-6"
        >
          <div class="flex flex-col gap-2">
            <label
              for="accountSelect"
              class="text-sm text-neutral-700 dark:text-neutral-300"
            >
              Choose the account you want to recover
            </label>
            <account-recovery-account-select
              v-model="address"
              :accounts="accounts"
              :error="!!addressError"
              :messages="addressError ? ['Please select a valid account address'] : undefined"
              :disabled="isLoadingAccounts"
              @update:model-value="validateAddress"
            />
          </div>

          <div class="flex flex-col gap-4">
            <ZkButton
              class="w-full"
              :disabled="!isValidAddress"
              @click="handleAccountContinue"
            >
              Continue
            </ZkButton>

            <ZkButton
              type="secondary"
              class="w-full"
              @click="currentStep = 1"
            >
              Back
            </ZkButton>
          </div>
        </div>

        <account-recovery-passkey-generation-flow-root
          v-if="currentStep >= 3"
          v-model:step-title="stepTitle"
          v-model:current-step="currentStep"
          v-model:disabled-steps="disabledSteps"
          :starting-step="3"
          :account-address="address"
        />
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import type { Address } from "viem";
import { ref } from "vue";

import { AddressSchema } from "~/utils/schemas";

const { getGuardedAccounts, getGuardedAccountsInProgress } = useRecoveryGuardian();

definePageMeta({
  layout: "dashboard",
});

const currentStep = ref(1);
const guardianAddress = ref("");
const guardianAddressError = ref("");
const isValidGuardianAddress = ref(false);
const address = ref("" as Address);
const addressError = ref("");
const isValidAddress = ref(false);
const accounts = ref<Address[]>([]);
const isLoadingAccounts = ref(false);
const disabledSteps = ref<number[]>([]);
const stepTitle = ref("");

watchEffect(() => {
  if (currentStep.value === 1) {
    stepTitle.value = "Enter Guardian Address";
  } else if (currentStep.value === 2) {
    stepTitle.value = "Select Your Account";
  }
  // Rest of step titles are handled inside the flow component
});

const validateGuardianAddress = async () => {
  const result = AddressSchema.safeParse(guardianAddress.value);
  if (!result.success) {
    guardianAddressError.value = "Not a valid address";
    isValidGuardianAddress.value = false;
    return;
  }

  // Reset errors without enabling the Continue button just yet
  guardianAddressError.value = "";
  isValidGuardianAddress.value = false;

  const guardedAccounts = await getGuardedAccounts(result.data);
  if (guardedAccounts.length === 0) {
    guardianAddressError.value = "No accounts found for this guardian";
    isValidGuardianAddress.value = false;
    return;
  }

  accounts.value = [...guardedAccounts]; // Clone the array to avoid mutating the original
  isValidGuardianAddress.value = true;
};

const validateAddress = () => {
  const result = AddressSchema.safeParse(address.value);
  if (result.success) {
    addressError.value = "";
    isValidAddress.value = true;
  } else {
    addressError.value = "Not a valid address";
    isValidAddress.value = false;
  }
};

const handleGuardianContinue = () => {
  if (isValidGuardianAddress.value) {
    currentStep.value = 2;
  }
};

const handleAccountContinue = () => {
  if (isValidAddress.value) {
    currentStep.value = 3;
  }
};
</script>
