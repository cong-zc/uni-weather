<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style>
	/*每个页面公共css */
	@import './components/icon/icon.css';
	.back{
		background-image: url('./static/weather_backgruond.jpg');
		background-color:transparent;
		background-size: 100% 100%;
	}
</style>
