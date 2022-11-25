<template>
  <p>TXID: ed88eb962c5b204ab409e94e2cbb83168b5fb0a27d1fa7e1e7ec1070b3e9157e</p>
  <form @submit.prevent="handleBoost" class="checkout-form">
  <HelloWorld msg="Boost POW Vue 3 Demo" />
    <div>
      <label for="txid">TXID: </label>
      <input type="text" id="txid" v-model="form.txid" required />
    </div>
    <div>
      <label for="amount">Amount: </label>
      <input type="text" id="amount" v-model="form.amount" />
    </div>
    <div>
      <label for="tag">Tag: </label>
      <input type="text" id="tag" v-model="form.tag" />
    </div>
    <div>
      <button type="submit">Submit</button>
    </div>
  </form>
  <p>New boost pow job txid: {{powjobTXID}}</p>
  <p>Indexer Data Response: {{ indexerDataResponse }}</p>

  <div id="relayx-button"></div>

</template>

<script lang="ts">
import { defineComponent, reactive, ref } from 'vue'
import axios from 'axios'
import { Script } from 'bsv'
import HelloWorld from './components/HelloWorld.vue'

// handleBoost('yo');

export default defineComponent({
  name: 'App',
  components: {
    HelloWorld
  }, 
  setup() {
    const form = reactive({
      txid: "",
      amount: 0.05,
      tag: "gop-test", 
    });
    const powjobTXID = ref('');
    const indexerDataResponse = ref('')
    return {
      form,
      powjobTXID,
      indexerDataResponse
    };
  },
  methods: {
    async handleBoost(txid: any) {

      const currency = 'USD';
      // NOTE: TEST GOPNIK ORDER
      const url = `https://pow.co/api/v1/boostpow/ed88eb962c5b204ab409e94e2cbb83168b5fb0a27d1fa7e1e7ec1070b3e9157e/new?value=${this.form.amount}&currency=${currency}&tag=${tag}`;

      console.log('boostpow.job.build', { url });

      let { data } = await axios.get(url);

      console.log('boostpow.payment_request', data);

      const script = new Script(data.outputs[0].script);

      const amount = data.outputs[0].amount / 100000000;

      // NOTE: Sets the OP Return data necessary to create a Boost POW job
      try {
        const send = {
          opReturn: [
            'onchain',
            '18pPQigu7j69ioDcUG9dACE1iAN9nCfowr',
            'job',
            JSON.stringify({
              index: 0
            })
          ],
          amount,
          to: script.toASM(),
          currency: 'BSV'
        };

        console.log('relayx.send.params', send);

        // QUESTION: Why does relayone.send work but not relayone.render? 
        // QUESTION: Is the total amount I'm spending the "amount" + the 0.0218 cents? Is this an arbitrary amount/address? 
        const result = await relayone.send({
          outputs: [send, {
            currency: 'USD',
            amount: 0.001,
            to: '1KDs8eoFYZ4zKWqS279PfmwXsRZFJdqVVC'
          }]
        });

        console.log('relayx.send.result', result);

        console.log('RESULT', result);

        const { txid } = result;
        this.powjobTXID = txid

        console.log('TXID', txid);

        // Post the new boostpow job transaction to the indexer API at pow.co
        // QUESTION: For clarity, is this so that the pow network can "see" that a new job has been created/is available to be mined? 
        axios
          .get(`https://pow.co/api/v1/boost/jobs/${txid}`)
          .then(({ data }) => {
            // QUESTION: How long does it take for a job to be mined on average and how do I then get the pow associated with each job that has the "gop-test" tag? 
            console.log(`pow.co/api/v1/jobs/${result.txid}.result`, data);
            this.indexerDataResponse = data
          })
          .catch((error) => {
            console.error(`pow.co/api/v1/jobs/${result.txid}`, error);
          });

        console.log('relay.quote', result);
      } catch (error) {
        console.error('relayx', error);
      }
    }
  },
})
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
