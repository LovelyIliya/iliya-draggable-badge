<template>
	<view class="content">
		<iliya-draggable-badge :number="number" @badgeBoom="badgeBoom"></iliya-draggable-badge>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				number: 20
			}
		},
		onLoad() {

		},
		methods: {
			badgeBoom() {
				this.number = 0
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding-top: 50px;
	}
</style>