<template>
  <iframe
    :src="sandboxUrl"
    style="
      width: 100%;
      height: 500px;
      border: 0;
      border-radius: 4px;
      overflow: hidden;
    "
    title="ecstatic-swartz-y5neq"
    allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
    sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
  ></iframe>
</template>

<script>
  export default {
    props: ['sandboxUrl'],
    data() {
      return {
        sandboxUrl: this.sandboxUrl
      }
    }
  }
</script>