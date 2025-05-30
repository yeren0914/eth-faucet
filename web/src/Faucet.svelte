<script>
  import { onMount } from 'svelte';
  import { getAddress } from '@ethersproject/address';
  import { CloudflareProvider } from '@ethersproject/providers';
  import { setDefaults as setToast, toast } from 'bulma-toast';

  let input = null;
  let faucetInfo = {
    account: '0x0000000000000000000000000000000000000000',
    network: 'testnet',
    payout: 1,
    symbol: 'ETH',
    hcaptcha_sitekey: '',
  };

  let mounted = false;
  let hcaptchaLoaded = false;

  onMount(async () => {
    const res = await fetch('/api/info');
    faucetInfo = await res.json();
    mounted = true;
  });

  window.hcaptchaOnLoad = () => {
    hcaptchaLoaded = true;
  };

  $: document.title = `${faucetInfo.symbol} ${capitalize(
    faucetInfo.network,
  )} Faucet`;

  let widgetID;
  $: if (mounted && hcaptchaLoaded) {
    widgetID = window.hcaptcha.render('hcaptcha', {
      sitekey: faucetInfo.hcaptcha_sitekey,
    });
  }

  setToast({
    position: 'bottom-center',
    dismissible: true,
    pauseOnHover: true,
    closeOnClick: false,
    animate: { in: 'fadeIn', out: 'fadeOut' },
  });

  async function handleRequest() {
    let address = input;
    if (address === null) {
      toast({ message: 'input required', type: 'is-warning' });
      return;
    }

    if (address.endsWith('.eth')) {
      try {
        const provider = new CloudflareProvider();
        address = await provider.resolveName(address);
        if (!address) {
          toast({ message: 'invalid ENS name', type: 'is-warning' });
          return;
        }
      } catch (error) {
        toast({ message: error.reason, type: 'is-warning' });
        return;
      }
    }

    try {
      address = getAddress(address);
    } catch (error) {
      toast({ message: error.reason, type: 'is-warning' });
      return;
    }

    try {
      let headers = {
        'Content-Type': 'application/json',
      };

      if (hcaptchaLoaded) {
        const { response } = await window.hcaptcha.execute(widgetID, {
          async: true,
        });
        headers['h-captcha-response'] = response;
      }

      const res = await fetch('/api/claim', {
        method: 'POST',
        headers,
        body: JSON.stringify({
          address,
        }),
      });

      let { msg } = await res.json();
      let type = res.ok ? 'is-success' : 'is-warning';
      toast({ message: msg, type });
    } catch (err) {
      console.error(err);
    }
  }

  function capitalize(str) {
    const lower = str.toLowerCase();
    return str.charAt(0).toUpperCase() + lower.slice(1);
  }
</script>

<svelte:head>
  {#if mounted && faucetInfo.hcaptcha_sitekey}
    <script
      src="https://hcaptcha.com/1/api.js?onload=hcaptchaOnLoad&render=explicit"
      async
      defer
    ></script>
  {/if}
</svelte:head>

<main>
  <section class="hero is-info is-fullheight">
    <div class="container">
      <div class="column is-12">
        <div id="hcaptcha" data-size="invisible"></div>
        <article class="faucet-box">
          <article class='faucet-main'>
              <h2 class='faucet-title'>KASPLEX-L2 TESTNET FAUCET</h2>
              <h5 class='faucet-subtitle'>50 Bridged KAS/24 hrs</h5>
              <section class='faucet-check-box mt40'>
                  <section class='faucet-address-box'>
                      <div class='faucet-input-box'>
                          <input
                            bind:value={input}
                            class="input is-rounded"
                            type="text"
                            placeholder="Enter your address or ENS name"
                          />
                          <button
                          on:click={handleRequest}
                          class="button is-primary is-rounded"
                        >
                          Request
                        </button>
                      </div>
                  </section>
              </section>
              <section class='faucet-fq-box mb30'>
                  <h6 class='faucet-fq-tit'>FAQs</h6>
                  <div class='faucet-fq-item'>
                      <h6>How does it work?</h6>
                      <p>You can request 50 Kasplex L2 KAS every 24h without any authentication. To request funds, enter your EOA wallet address and click 'Send Me KAS.' Smart contract addresses are not supported.</p>
                      <p>I get an error saying "To receive L2 KAS, your wallet must have a balance of less than 2 KAS on Kasplex L2." </p>
                      <p>This faucet is built to supply test tokens to developers for testing smart contracts and engaging with Kasplex L2.A balance exceeding 2 KAS is enough for testing purposes.</p>
                  </div>
              </section>
          </article>
      </article>
      </div>
    </div>
  </section>
</main>

<style>
  .hero.is-info {
    background: url('/faucet-bg.png') no-repeat center -30px;
    -webkit-background-size: cover;
    -moz-background-size: cover;
    -o-background-size: cover;
    background-size: cover;
  }
  .navbar-brand:hover  {
    background-color: #008F8D;
  }
  .hero .subtitle {
    line-height: 1.5;
  }
  .box {
    border-radius: 19px;
  }
  .top-box {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    height: 60px;
    line-height: 60px;
    padding: 0 50px;
    padding-left: 0;
    border-bottom: 2px solid rgba(0, 80, 85, .5);
    background-color: #000000;
    z-index: 100;

}
.logo-box {
    position: absolute;
    left: 50px;
  }
.logo-box img {
    width: 110px;
    vertical-align: middle;
    cursor: pointer;
}
  .faucet-box {
  position: relative;
  width: 100%;
  height: auto;
  min-height: 100vh;
  padding-top: 40px;
  padding-bottom: 25px;
  background: url('../assets/faucet-bg.png') no-repeat center 0px;
  background-size: cover;
}

.faucet-top {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  width: 100%;
  height: 40px;
  padding: 12px 30px;
  box-sizing: border-box;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-image: linear-gradient(to right, #073237, #000E0F, #042A2D);
}

.faucet-top .faucet-top-left strong {
  display: inline-block;
  margin-right: 10px;
  font-size: 13px;
  font-weight: bold;
  word-break: break-word;
  color: #ffffff;
}

.faucet-top .faucet-top-left .faucet-logo {
  display: inline-block;
  vertical-align: middle;
  width: 88px;
}

.faucet-top .faucet-top-left .faucet-round {
  margin-left: 10px;
  display: inline-block;
  background-color: #D36300;
  width: 9px;
  height: 9px;
  border-radius: 50%;
}

.faucet-input-box {
  display: flex;
  justify-content: space-between;
  gap: 10px;
}

.faucet-input-box input:focus {
  box-shadow: none;
}

.btn-top-primary {
  background-color: #4DA9A6;
  border-radius: 4px;
  padding: 6px 13px;
  box-sizing: border-box;
  color: #ffffff;
  font-size: 12px;
  border: none;
  cursor: pointer;
}

.btn-top-primary::after {
  display: inline-block;
  position: relative;
  content: "";
  width: 1px;
  height: 10px;
  margin-left: 10px;
  background-color: #ffffff;
}

.btn-top-primary:hover {
  background-color: #449f9c !important;
}

.faucet-main {
  max-width: 840px;
  margin: 0 auto;
  padding: 0 20px;
  box-sizing: border-box;
}

.faucet-main .faucet-title {
  margin-top: 80px;
  margin-bottom: 20px;
  font-size: 38px;
  font-weight: bold;
  text-align: center;
}

.faucet-main .faucet-subtitle {
  text-align: center;
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 30px;
}

.faucet-main .faucet-subtitle-box {
  margin: 30px 0 20px;
  padding: 15px 36px;
  box-sizing: border-box;
  text-align: center;
  font-size: 14px;
  line-height: 24px;
  color: #ffffff;
  font-weight: 600;
  background-image: linear-gradient(to right, #005E61, #008F8D);
  font-family: 'Lato-Semibold';
}

.faucet-main .faucet-check-box {
  margin-bottom: 58px;
  background-color: #ffffff;
  border-radius: 20px;
  padding: 30px 40px;
  box-sizing: border-box;
}

.faucet-main .ant-btn-primary {
  background-color: #018382;
  box-shadow: none;
  border: none;
  color: #ffffff;
  font-weight: bold;
}

.faucet-main .faucet-check-google {
  margin-top: 20px;
  color: #001F23;
}

.faucet-main .faucet-fq-box {
  background-color: rgba(255, 255, 255,.8);
  border-radius: 30px;
  padding: 40px 50px 20px;
  box-sizing: border-box;
}

.faucet-main .faucet-fq-box .faucet-fq-tit {
  font-size: 25px;
  font-weight: bold;
  color: #000000;
  margin-bottom: 20px;
}

.faucet-main .faucet-fq-box .faucet-fq-item {
  margin-bottom: 40px;
  word-break: break-word;
}

.faucet-main .faucet-fq-box .faucet-fq-item h6 {
  margin-bottom: 10px;
  font-size: 16px;
  color: #2B2A2A;
  font-weight: bold;
}

.faucet-main .faucet-fq-box .faucet-fq-item p {
  margin-bottom: 20px;
  font-size: 15px;
  color: #565656;
  line-height: 22px;
}

.faucet-main .faucet-fq-box .faucet-fq-item a {
  color: #13A1AD;
  cursor: pointer;
}

.faucet-main .faucet-fq-box .faucet-fq-item .flex-p {
  display: flex;
  align-items: center;
}

.faucet-main .faucet-fq-box .faucet-fq-item strong {
  margin-left: 10px;
  background-color: #000;
  border-radius: 20px;
  padding: 0 8px;
  box-sizing: border-box;
  font-size: 12px;
  color: #ffffff;
  min-width: 67px;
  font-weight: 500;
  display: flex;
  justify-content: center;
  align-items: center;
}

.faucet-main .faucet-fq-box .faucet-fq-item strong img {
  display: inline-block;
  margin-right: 6px;
  width: 12px;
}

.faucet-main .faucet-fq-box .faucet-fq-address {
  display: inline-block;
  padding: 9px 14px;
  box-sizing: border-box;
  border-radius: 6px;
  color: #333333;
  font-size: 14px;
  word-break: break-all;
  background-color: #ffffff;
}

.faucet-copy {
  margin-top: 80px;
  font-size: 14px;
  color: #ffffff;
  text-align: center;
}

@media screen and (max-width: 768px) {
  .faucet-top {
    padding: 10px 20px;
  }
  .btn-top-primary {
    padding: 3px 6px;
  }
  .faucet-main .faucet-title {
    font-size: 28px;
  }
  .faucet-main .faucet-subtitle {
    font-size: 16px;
  }
  .faucet-main .faucet-check-box {
    padding: 15px 20px;
  }
  .faucet-main .faucet-fq-box {
    padding: 30px 20px 20px;
  }
}

</style>
