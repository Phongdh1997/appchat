<template>
	<view :style="{flex: 1, justifyContent: 'center'}">
    	<activity-indicator size="large" color="#0000ff" />
    	<text :style="{textAlign: 'center'}">Loading...</text>
    </view>
</template>

<script>
	export default {
		name: 'LoadScreen',
		props: {
			opacity: {
				type: Boolean,
				default: false
			}
		}
  };

</script>

<style>

</style>