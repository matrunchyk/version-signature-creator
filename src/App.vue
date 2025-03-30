<template>
  <div class="bg-white">
    <div class="mx-auto max-w-7xl px-4 py-24 sm:px-6 sm:py-20 lg:px-8">
      <div class="mx-auto max-w-2xl">
        <form>
          <div class="space-y-12">
            <div class="border-b border-gray-900/10 pb-12">
              <div class="flex items-center gap-x-4">
                <img src="./assets/logo.png" alt="Version Signature Generator" class="h-12 w-auto"/>
                <h2 class="text-base/7 font-semibold text-gray-900">
                  Version Signature Generator
                </h2>
              </div>
              <p class="text-sm/6 text-gray-600">This app helps you to generate a version signature from service
                versions.</p>
              <div class="mt-10 grid grid-cols-1 gap-x-6 gap-y-8 sm:grid-cols-6">
                <div class="col-span-full">
                  <label for="json" class="block text-sm/6 font-medium text-gray-900">
                    JSON
                  </label>
                  <div class="mt-2">
                    <textarea
                        ref="textarea"
                        id="json"
                        name="json"
                        v-model="input"
                        rows="10"
                        :class="textareaClasses"
                    ></textarea>
                  </div>
                </div>

                <div class="sm:col-span-4">
                  <label for="signature" class="block text-sm/6 font-medium text-gray-900">Signature hash</label>
                  <div class="mt-2">
                    <input id="signature" name="signature" type="text" readonly
                           class="block w-full rounded-md bg-gray-100 px-3 py-1.5 text-base text-gray-900 outline-1 -outline-offset-1 outline-gray-300 placeholder:text-gray-400 focus:outline-2 focus:-outline-offset-2 focus:outline-indigo-600 sm:text-sm/6"
                           :value="calculateMD5(input)"/>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="mt-6 flex items-center justify-end gap-x-6" v-if="isSupported">
            <button @click="copy(signature)" type="button"
                    class="inline-flex cursor-pointer items-center rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-xs hover:bg-indigo-500 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">
              <span v-if="!copied">Copy to Clipboard</span>
              <template v-else>
                <svg class="mr-1.5 -ml-0.5 size-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true"
                     data-slot="icon">
                  <path fill-rule="evenodd"
                        d="M16.704 4.153a.75.75 0 0 1 .143 1.052l-8 10.5a.75.75 0 0 1-1.127.075l-4.5-4.5a.75.75 0 0 1 1.06-1.06l3.894 3.893 7.48-9.817a.75.75 0 0 1 1.05-.143Z"
                        clip-rule="evenodd"></path>
                </svg>
                Copied!
              </template>
            </button>
            <button type="button" class="mt-3 cursor-pointer inline-flex w-full justify-center rounded-md bg-white px-3 py-2 text-sm font-semibold text-gray-900 ring-1 shadow-xs ring-gray-300 ring-inset hover:bg-gray-50 sm:mt-0 sm:w-auto" @click="input = ''">Clear</button>
          </div>
        </form>
      </div>
    </div>
    <div class="absolute bottom-4 text-center w-full text-sm text-gray-400">
      Made with 💜 for best DevOps
    </div>
  </div>
</template>

<script setup lang="ts">
import md5 from 'md5';
import { useFocus, useClipboard, useTextareaAutosize } from '@vueuse/core'
import { computed, ref } from "vue";

const {textarea, input} = useTextareaAutosize({styleProp: 'minHeight'})
const {copy, copied, isSupported} = useClipboard({source: input})
const signature = ref('');
const isValid = ref(false);

useFocus(textarea, {initialValue: true})

type ServiceVersions = Record<string, string>;

const textareaClasses = computed(() => {
  return [
    'block w-full rounded-md bg-gray-100 px-3 py-1.5 text-base text-gray-900 outline-1 -outline-offset-1 placeholder:text-gray-400 focus:outline-2 focus:-outline-offset-2 sm:text-sm/6',
    {'border-gray-300 focus:outline-indigo-600': isValid.value || !input.value},
    {'border-red-500 focus:outline-red-600': !isValid.value && input.value},
  ]
})

function parseJSON(string: string) {
  try {
    const parsed = JSON.parse(string);
    if (typeof parsed === 'object' && parsed !== null) {
      isValid.value = true;

      return parsed as ServiceVersions;
    } else {
      console.error('Parsed value is not an object');
      isValid.value = false;
      return null;
    }
  } catch (error) {
    console.error('Error parsing JSON:', error);
    isValid.value = false;
    return null;
  }
}

function getServiceVersions(string: string): ServiceVersions | null {
  if (string) {
    // Remove all whitespace characters
    string = string.replace(/\s+/g, '');

    // Check if the string is a valid JSON object
    if (string.startsWith('{') && string.endsWith('}')) {
      return parseJSON(string);
    }

    isValid.value = false;
  }

  return null;
}

function calculateMD5(servicesVersion: string): string {
  const serviceVersions = getServiceVersions(servicesVersion);

  if (!serviceVersions) {
    console.warn('Invalid JSON format');
    return '';
  }

  const versionString = Object.keys(serviceVersions)
      .sort((a, b) => {
        if (a > b) return 1;
        if (b > a) return -1;

        return 0;
      })
      .map(serviceKey => `${serviceKey}=${serviceVersions[serviceKey]}`)
      .join(',');

  return md5(versionString);
}

</script>
