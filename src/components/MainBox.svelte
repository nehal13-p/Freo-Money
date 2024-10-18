<script lang="ts">
  import { onMount } from 'svelte';

  export let addNotification: (message: string) => void;

  let phone = '';
  let otp = ['', '', '', ''];
  let whatsapp = true;
  let otpSent = false;
  let loading = false;
  let otpTimer = 30;
  let canResendOtp = false;
  let showPhoneBox = true;
  let showOtpBox = false;

  const APP_RECAPTCHA = '6LciytgUAAAAAG0vYzXVgh7ZTZgTYPlC_SI_YrFL';

  onMount(() => {
    // @ts-ignore
    window.grecaptcha.ready(() => {
      // Recaptcha is ready
    });
  });

  function handleOnChangeOfPhone(e: Event) {
    const value = (e.target as HTMLInputElement).value;
    const reg = /^\d+$/;
    if (!reg.test(value)) {
      phone = '';
    } else {
      phone = value;
    }
    if (value.length === 10) {
      handleSendOtp();
    }
  }

  function handleSendOtp() {
    if (phone && phone.length === 10) {
      startLoading();
      const headers = getHeaders();

      // @ts-ignore
      window.grecaptcha.execute(APP_RECAPTCHA, { action: 'auth_qual_2' })
        .then((token: string) => {
          const rawBody = { phone, purpose: 'PH_VERIFICATION', token };
          const requestOptions = {
            method: "POST",
            headers,
            body: JSON.stringify(rawBody),
          };

          fetch("https://app.moneytap.com/v1/otp/send", requestOptions)
            .then((response) => {
              if (response && response.ok) {
                otpSent = true;
                showPhoneBox = false;
                showOtpBox = true;
                startOtpTimer();
              } else {
                addNotification('Something Went Wrong');
              }
              stopLoading();
            })
            .catch(() => {
              stopLoading();
              addNotification('Something Went Wrong');
            });
        });
    } else {
      addNotification('Please enter correct mobile number');
    }
  }

  function startOtpTimer() {
    otpTimer = 30;
    canResendOtp = false ;
    const timer = setInterval(() => {
      otpTimer--;
      if (otpTimer === 0) {
        canResendOtp = true;
        clearInterval(timer);
      }
    }, 1000);
  }

  function getAccessTokenAndRedirect() {
    if (otp.join('').length === 4) {
      startLoading();
      const authHeader = btoa(`moneytap-webqual:Aewaeshi3phoo7Kooc6ahDu0Oolei0th`);
      const headers = new Headers({
        'Authorization': `Basic ${authHeader}`,
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8',
        'app-platform': 'ooneipiDei0Tequ0Thaepheech8ietoh',
        'mt-request-id': generateUUID()
      });

      const requestOptions = {
        method: "POST",
        headers,
        body: `grant_type=password&username=${phone}&scope=read+write&password=${otp.join('')}&auth_type=OTP&auth_version=2`
      };

      fetch('https://app.moneytap.com/oauth/token', requestOptions)
        .then(async (response) => {
          const res = await response.json();
          if (response.ok && response.status ===  200 && res. access_token) {
            captureConsent(res.access_token);
          } else {
            addNotification('Unable to verify credentials.');
          }
          stopLoading();
        })
        .catch(() => {
          stopLoading();
          addNotification('Something Went Wrong');
        });
    }
  }

  function captureConsent(token: string) {
    const payload = {
      tcAccepted: true,
      tcAcceptedAt: new Date().toISOString(),
      whatsAppOptIn: whatsapp ? 'YES' : 'NO',
      privacyPolicyAccepted: true,
      privacyPolicyAcceptedAt: new Date().toISOString(),
    };

    const headers = getHeaders();
    headers.set('Authorization', `Bearer ${token}`);

    fetch("https://app.moneytap.com/customer/consent", {
      method: 'POST',
      headers,
      body: JSON.stringify(payload),
    })
      .then((response) => {
        if (response && response.ok) {
          window.location.href = `https://web.moneytap.com/app/?token=${token}`;
        }
      })
      .catch(() => {
        addNotification('Something Went Wrong');
      });
  }

  function startLoading() {
    loading = true;
  }

  function stopLoading() {
    loading = false;
  }

  function getHeaders() {
    const myHeaders = new Headers();
    myHeaders.append("sec-ch-ua", "\"Chromium\";v=\"128\", \"Not;A=Brand\";v=\"24\", \"Google Chrome\";v=\"128\"");
    myHeaders.append("Pragma", "no-cache");
    myHeaders.append("sec-ch-ua-mobile", "?0");
    myHeaders.append("app-platform", "ooneipiDei0Tequ0Thaepheech8ietoh");
    myHeaders.append("Content-Type", "application/json");
    myHeaders.append("Accept", "application/json");
    myHeaders.append("mt-request-id", generateUUID());
    myHeaders.append("Cache-Control", "no-cache, no-store");
    return myHeaders;
  }

  function generateUUID() {
    // @ts-ignore
    var d = new Date().getTime();
    var d2 = ((typeof performance !== 'undefined') && performance.now && (performance.now() * 1000)) || 0; 
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
      var r = Math.random() * 16; 
      if (d > 0) { 
        r = (d + r) % 16 | 0;
        d = Math.floor(d / 16);
      } else { 
        r = (d2 + r) % 16 | 0;
        d2 = Math.floor(d2 / 16);
      }
      return (c === 'x' ? r : (r & 0x3 | 0x8)).toString(16);
    });
  }
</script>

<div class="main-box">
  {#if showPhoneBox}
    <div class="box show" id="phone-box">
      <label>Enter your <b>mobile number</b> and <br /> Get your <b>money</b></label>
      <input placeholder="+91" id="phone-number" maxlength="10" on:input={handleOnChangeOfPhone} />
      <p class="text-small">An OTP will be sent for verification </p>
      <button on:click={handleSendOtp} id="send-otp-btn">Send OTP</button>
    </div>
  {/if}
  {#if showOtpBox}
    <div class="box show" id="otp-box">
      <label>Enter your OTP Get your money</label>
      <div class="otp-row" id="otp-row">
        {#each otp as digit, index}
          <input maxlength="1" id={`otp-${index + 1}`} type="text" bind:value={digit} />
        {/each}
      </div>
      {#if otpSent}
        <p style="padding: 0; margin: 0; font-size: 10px;" id="otp-sent-text">Otp has been sent to <b>{phone}</b></p>
      {/if}
      <button on:click={getAccessTokenAndRedirect} id="submit-otp-btn">Submit OTP</button>
      {#if canResendOtp}
        <p on:click={ handleSendOtp} style="padding: 0; margin: 0; font-size: 10px; text-decoration: underline; cursor: pointer;" id="resend-otp-btn">Resend OTP</p>
      {/if}
      {#if !canResendOtp}
        <p style="padding: 0; margin: 0; font-size: 10px;" id="timer">Resend in {otpTimer}s</p>
      {/if}
    </div>
  {/if}
  {#if loading}
    <div class="box show" id="loading-box">
      <div class="loader"></div>
    </div>
  {/if}
  <div class="tnc-container">
    <div class="tnc">
      <input id="whatsapp" type="checkbox" bind:checked={whatsapp} />
      <label for="whatsapp">Opt for WhatsApp Updates</label>
    </div>
    <div class="tnc">
      <input id="terms-and-conditions" type="checkbox" />
      <label for="terms-and-conditions">By pressing 'Continue', you agree to Freo's <a href="https://www.moneytap.com/terms-and-policies.html">Terms & Policies</a>. You also agree to <a href="https://www.moneytap.com/consumer-connect.pdf">CIBIL Terms & Conditions</a> and allow MWYN TECH's Partners to access my credit report stored in Credit bureaus.</label>
    </div>
  </div>
</div>

<style>
  .main-box {
    background-color: white;
    display: flex;
    flex-direction: column;
    order: 1;
    justify-content: start;
    padding: 10px;
    padding-top: 20px;
    color: black;
    width: 92%;
    max-width: 809px;
    margin: 0 4.4%;
    position: absolute;
    top: 50%;
    border-radius: 20px;
    font-family: "Poppins", system-ui;
    min-height: 300px;
    box-sizing: border-box;
  }

  .box {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .box label {
    font-size: 14px;
  }

  .box input {
    padding: 10px;
    border-radius: 27px;
    background-color: white;
    border: 1px solid rgb(178, 176, 176);
    color: black;
    outline: none;
  }

  .box button {
    padding: 10px;
    border-radius: 27px;
    background-color: rgb(252, 81, 6);
    border: none;
    color: #fff;
    cursor: pointer;
  }

  .text-small {
    font-size: 12px;
    padding-left: 0;
  }

  .otp-row {
    display: flex;
    gap: 10px;
  }

  .otp-row input {
    width: 20px;
    color: black;
    text-align: center;
  }

  .loader {
    width: 45px;
    aspect-ratio: 0.75;
    --c: no-repeat linear-gradient(#ff822e 0 0);
    background: var(--c) 0% 50%, var(--c) 50% 50%, var(--c) 100% 50%;
    animation: l7 1s infinite linear alternate;
    margin: auto;
  }

  @keyframes l7 {
    0% { background-size: 20% 50%, 20% 50%, 20% 50% }
    20% { background-size: 20% 20%, 20% 50%, 20% 50% }
    40% { background-size: 20% 100%, 20% 20%, 20% 50% }
    60% { background-size: 20% 50%, 20% 100%, 20% 20% }
    80% { background-size: 20% 50%, 20% 50%, 20% 100% }
    100% { background-size: 20% 50%, 20% 50%, 20% 50% }
  }

  .tnc-container {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: 10px;
  }

  .tnc {
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .tnc input[type="checkbox"] {
    appearance: none;
    -webkit-appearance: none;
    width: 20px;
    height: 20px;
    border: 2px solid #a89f9f;
    border-radius: 7px;
    background-color: transparent;
    position: relative;
    margin: 0;
    cursor: pointer;
    flex-shrink: 0;
  }

  .tnc input[type="checkbox"]:checked::before {
    content: "\2713";
    font-size: 12px;
    color: black;
    position: absolute;
    top: 50%; left: 50%;
    transform: translate(-50%, -50%);
  }

  .tnc label {
    font-size: 10px;
    color: black;
    cursor: pointer;
    line-height: 1.2;
    margin: 0;
  }

  .tnc a {
    color: #3747FE;
    text-decoration: none;
  }

  @media screen and (min-width: 768px) {
    .main-box {
      margin: auto;
      padding: 20px 100px;
      max-width: unset;
      width: 950px;
      align-self: center;
      border-radius: 40px;
    }

    .main-box .box {
      align-self: center;
      width: 400px;
    }

    .tnc-container {
      align-self: center;
      max-width: 400px;
    }
  }
</style>