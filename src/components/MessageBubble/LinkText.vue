<template>
<div class="qkb-msg-bubble-component qkb-msg-bubble-component--single-text">
    <div class="qkb-msg-bubble-component__text"><a :href="mainData.text" target="_blank">{{ mainData.text }}</a></div>
</div>
</template>
<script>
export default {
  props: {
    mainData: {
      type: Object
    }
  }
}
</script>
