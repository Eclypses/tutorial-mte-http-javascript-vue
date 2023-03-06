<!--
THIS SOFTWARE MAY NOT BE USED FOR PRODUCTION. Otherwise,
The MIT License (MIT)

Copyright (c) Eclypses, Inc.

All rights reserved.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
-->

<template>
  <div class="m-4">
    <div class="container bg-light rounded p-4">
      <div class="row">
        <h1 class="display-6 ml-3">Vue HTTP MTE Tutorial</h1>
        <hr />
      </div>

      <div class="row">
        <form>
          <div class="row">
            <div class="ml-3 col-4">
              <label class="form-label"> IP </label>
              <input
                type="text"
                class="form-control"
                placeholder="localhost"
                v-model="ip"
              />
            </div>

            <div class="col-4">
              <label class="form-label"> Port </label>
              <input
                type="text"
                class="form-control"
                placeholder="27015"
                v-model="port"
              />
            </div>
          </div>
        </form>
      </div>

      <div class="col-md-12 mt-4">
        <hr />
      </div>

      <form @submit.prevent="onSubmit" class="row g-3">
        <div class="col-12">
          <label class="form-label"> Text To Send (Send 'quit' to exit) </label>
          <input type="text" class="form-control" v-model="message" />
        </div>

        <div class="col-12 mt-4">
          <label class="form-label">Encoded Text Sent To HTTP Server</label>
          <input
            type="text"
            class="form-control"
            v-model="outgoingEncodedMessage.str"
            disabled
          />
        </div>

        <div class="col-12 mt-4">
          <label class="form-label">
            Encoded Text Received From HTTP Server
          </label>
          <input
            type="text"
            class="form-control"
            v-model="incomingEncodedMessage"
            disabled
          />
        </div>

        <div class="col-12 mt-4">
          <label class="form-label">
            Decoded Text Received From HTTP Server
          </label>
          <input
            type="text"
            class="form-control"
            v-model="decodedMessage.str"
            disabled
          />
        </div>

        <div class="row mt-4">
          <div class="ml-3 col">
            <button type="submit" class="btn btn-primary">Send</button>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onBeforeMount, ref } from 'vue';
import { fromByteArray } from 'base64-js';
import 'bootstrap/dist/css/bootstrap.min.css';
/* Step 2 */
import {
  MteDec,
  MteEnc,
  MteStatus,
  MteBase,
  MteWasm,
  type MteStrStatus,
  // MteMkeDec,          // Uncomment to use MKE
  // MteMkeEnc,          // Uncomment to use MKE
  // MteFlenEnc,         // Uncomment to use MTEFLEN
} from './Mte';

export default defineComponent({
  name: 'App',
  setup() {
    // This tutorial uses HTTP for communication.
    // It should be noted that the MTE can be used with any type of communication. (HTTP is not required!)

    const ip = ref('');
    const port = ref('');
    const message = ref('');
    const outgoingEncodedMessage = ref<MteStrStatus>({
      str: '',
      status: MteStatus.mte_status_success,
    });
    const incomingEncodedMessage = ref('');
    const decodedMessage = ref<MteStrStatus>({
      str: '',
      status: MteStatus.mte_status_success,
    });

    /* Step 3 */
    // Add default state for MTE properties that we need global access to
    const wasm = ref<MteWasm>();
    const base = ref<MteBase>();
    const decoderStatus = ref(MteStatus.mte_status_success);
    const encoderStatus = ref(MteStatus.mte_status_success);

    //---------------------------------------------------
    // Comment out to use MKE or MTE FLEN instead of MTE Core
    //---------------------------------------------------
    const encoder = ref<MteEnc>();
    const decoder = ref<MteDec>();

    //---------------------------------------------------
    // Uncomment to use MKE instead of MTE Core
    //---------------------------------------------------
    // const decoder = ref<MteMkeDec>();
    // const encoder = ref<MteMkeEnc>();

    //---------------------------------------------------
    // Uncomment to use MTE FLEN instead of MTE Core
    //---------------------------------------------------
    // const fixedLength = 8;
    // const encoder = ref<MteFlenEnc>();
    // const decoder = ref<MteDec>();

    // /* Step 4 */
    // Instantiate the MteWasm and MteBase
    const instantiateMte = async () => {
      wasm.value = new MteWasm();
      await wasm.value.instantiate();
      base.value = new MteBase(wasm.value);
    };

    /* Step 5 */
    // Create function to run MTE tests
    const runMteTests = () => {
      if (base.value) {
        const licenseCompany = 'Eclypses Inc.';
        const licenseKey = 'Eclypses123';

        // Check MTE license
        // Initialize MTE license. If a license code is not required (e.g., trial mode), this can be skipped.
        if (!base.value.initLicense(licenseCompany, licenseKey)) {
          const licenseStatus = MteStatus.mte_status_license_error;

          console.error(
            `License error (${base.value.getStatusName(
              licenseStatus,
            )}): ${base.value.getStatusDescription(licenseStatus)}`,
          );
        }
      }
    };

    /* Step 6 */
    // Create Encoder
    const createEncoder = () => {
      if (wasm.value && base.value) {
        //---------------------------------------------------
        // Comment out to use MKE or MTE FLEN instead of MTE Core
        //---------------------------------------------------
        const mteEncoder = MteEnc.fromdefault(wasm.value);

        //---------------------------------------------------
        // Uncomment to use MKE instead of MTE Core
        //---------------------------------------------------
        // const mteEncoder = MteMkeEnc.fromdefault(wasm.value);

        //---------------------------------------------------
        // Uncomment to use MTE FLEN instead of MTE Core
        //---------------------------------------------------
        // const mteEncoder = MteFlenEnc.fromdefault(wasm.value, fixedLength.value);

        const entropyMinBytes = base.value.getDrbgsEntropyMinBytes(
          mteEncoder.getDrbg(),
        );

        const encoderEntropy =
          entropyMinBytes > 0 ? '0'.repeat(entropyMinBytes) : '';
        const encoderNonce = '1';
        const encoderPersonalizationString = 'demo';

        // Set entropy and nonce for the encoder
        mteEncoder.setEntropyStr(encoderEntropy);
        mteEncoder.setNonce(encoderNonce);

        // Initialize MTE encoder with personalization string
        encoderStatus.value = mteEncoder.instantiate(
          encoderPersonalizationString,
        );

        if (encoderStatus.value !== MteStatus.mte_status_success) {
          console.error(
            `Failed to initialize the MTE encoder engine.  Status: ${base.value.getStatusName(
              encoderStatus.value,
            )} / ${base.value.getStatusDescription(encoderStatus.value)}`,
          );
        } else {
          encoder.value = mteEncoder;
        }
      }
    };

    /* Step 6 CONTINUED... */
    // Create Decoder
    const createDecoder = () => {
      if (wasm.value && base.value) {
        //---------------------------------------------------
        // Comment out to use MKE instead of MTE Core
        //---------------------------------------------------
        const mteDecoder = MteDec.fromdefault(wasm.value);

        //---------------------------------------------------
        // Uncomment to use MKE instead of MTE Core
        //---------------------------------------------------
        // const mteDecoder = MteMkeDec.fromdefault(wasm.value);

        const entropyMinBytes = base.value.getDrbgsEntropyMinBytes(
          mteDecoder.getDrbg(),
        );

        const decoderEntropy =
          entropyMinBytes > 0 ? '0'.repeat(entropyMinBytes) : '';
        const decoderNonce = '0';
        const decoderPersonalizationString = 'demo';

        // Set entropy and nonce for the encoder
        mteDecoder.setEntropyStr(decoderEntropy);
        mteDecoder.setNonce(decoderNonce);

        // Initialize MTE encoder with personalization string
        decoderStatus.value = mteDecoder.instantiate(
          decoderPersonalizationString,
        );

        if (decoderStatus.value !== MteStatus.mte_status_success) {
          console.error(
            `Failed to initialize the MTE encoder engine.  Status: ${base.value.getStatusName(
              decoderStatus.value,
            )} / ${base.value.getStatusDescription(decoderStatus.value)}`,
          );
        } else {
          decoder.value = mteDecoder;
        }
      }
    };

    const onSubmit = async () => {
      /* Step 7 */
      // Encode text to send and ensuring successful
      if (encoder.value && base.value) {
        outgoingEncodedMessage.value = encoder?.value?.encodeStrB64(
          message.value,
        );

        if (
          outgoingEncodedMessage.value.status !== MteStatus.mte_status_success
        ) {
          console.error(
            `Error encoding: Status: ${base.value.getStatusName(
              outgoingEncodedMessage.value.status,
            )} / ${base.value.getStatusDescription(
              outgoingEncodedMessage.value.status,
            )}`,
          );
        }
      }

      if (!ip.value) ip.value = 'localhost';
      if (!port.value) port.value = '27015';

      // Send message over HTTP and receive response
      const response = await fetch(`http://${ip.value}:${port.value}/echo`, {
        method: 'POST',
        mode: 'cors',
        body: outgoingEncodedMessage.value?.str,
      });

      // Grab byte array out of response
      const encodedMessage = await response.arrayBuffer();
      const byteArray = new Uint8Array(encodedMessage);

      incomingEncodedMessage.value = fromByteArray(byteArray);

      /* Step 7 */
      // Decode incoming message and check for successful response
      if (decoder.value && base.value) {
        decodedMessage.value = decoder.value.decodeStr(byteArray);

        if (decodedMessage.value.status !== MteStatus.mte_status_success) {
          console.log(
            `Error decoding: Status: ${base.value.getStatusName(
              decodedMessage.value.status,
            )} / ${base.value.getStatusDescription(
              decodedMessage.value.status,
            )}`,
          );
        }
      }

      message.value = '';
    };

    /* Step 8 */
    // Call MTE methods at the start of the application with Vue's lifecycle hook
    onBeforeMount(async () => {
      await instantiateMte();
      runMteTests();
      createEncoder();
      createDecoder();
    });

    return {
      ip,
      port,
      message,
      outgoingEncodedMessage,
      incomingEncodedMessage,
      decodedMessage,
      onSubmit,
    };
  },
});
</script>
