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
            v-model="outgoingEncodedMessage"
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
            v-model="decodedMessage"
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
import { defineComponent, ref } from 'vue';
import { fromByteArray } from 'base64-js';
import 'bootstrap/dist/css/bootstrap.min.css';

export default defineComponent({
  name: 'App',
  setup() {
    // This tutorial uses HTTP for communication.
    // It should be noted that the MTE can be used with any type of communication. (HTTP is not required!)

    const ip = ref('');
    const port = ref('');
    const message = ref('');
    const outgoingEncodedMessage = ref('');
    const incomingEncodedMessage = ref('');
    const decodedMessage = ref('');

    const onSubmit = async () => {
      // MTE Encoding the text (into string to be consistent with other tutorial labs)
      // would go here prior to sending over HTTP

      outgoingEncodedMessage.value = message.value;
      if (!ip.value) ip.value = 'localhost';
      if (!port.value) port.value = '27015';

      // Send message over HTTP and receive response
      const response = await fetch(`http://${ip.value}:${port.value}/echo`, {
        method: 'POST',
        body: message.value,
      });

      // Grab byte array out of response
      const encodedMessage = await response.arrayBuffer();
      const byteArray = new Uint8Array(encodedMessage);

      incomingEncodedMessage.value = fromByteArray(byteArray);

      // MTE Decoding the bytes would go here instead of using the Node TextDecoder
      decodedMessage.value = new TextDecoder().decode(byteArray);

      message.value = '';
    };

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
