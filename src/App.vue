<script>
import feather from 'feather-icons';
import AppHeaderLogin from './components/shared/AppHeaderLogin';
import AppFooter from './components/shared/AppFooter';
import AppHeaderGuest from './components/shared/AppHeaderGuest';
// import Login from './views/Login';
import firebaseApp from './firebase.js';
import { getAuth, onAuthStateChanged, signOut } from "firebase/auth";

export default {
	components: {
		AppHeaderLogin,
		AppFooter,
		AppHeaderGuest,
		// Login,
	},
	data: () => {
		return {
			appTheme: localStorage.getItem('theme'),
			display: 2,
			dontshow: false,
		};
	},
	mounted() {
		feather.replace();
		const auth = getAuth(firebaseApp);
		onAuthStateChanged(auth, (user) => {
			if (user) {
				this.display = 1;
			} else {
				this.display = 2
			}
		})
		this.$router.push('/Home')
	},
	updated() {
		feather.replace();
	},
	methods: {
		goToLogin() {
			this.$router.push('/Login');
		},
		logoutSuccess() {
			console.log('test')
			const auth = getAuth(firebaseApp);
			const user = auth.currentUser;
			signOut(auth, user)
			this.$router.push('/Home')
		}
		// loginSuccess() {
		// 	this.display = 2;
		// 	this.$router.push('/Home');
		// },
		// logoutSuccess() {
		// 	this.display = 2;
		// 	this.$router.push('/Home');
		// } 
	}
};
</script>

<template>
	<div :class="appTheme" class="pt-0.5">
		<!-- App header -->
		<div v-show = "display == 1"><AppHeaderLogin @logout-update="logoutSuccess()"/></div>
		<div v-show = "display == 2"><AppHeaderGuest @call-login="goToLogin()"/></div>
		<!-- <div v-show="dontshow"><Login @login-success="loginSuccess()"/></div> -->

		<!-- Render active component contents with vue transition -->
		<transition name="fade" mode="out-in">
			<router-view :theme="appTheme" />
		</transition>

	
		<!-- Scroll to top -->
		<back-to-top
			visibleoffset="500"
			right="30px"
			bottom="20px"
			class="shadow-lg"
		>
			<i data-feather="chevron-up"></i>
		</back-to-top>

		<!-- App footer -->
		<AppFooter />
	</div>
</template>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
}

.vue-back-to-top {
	@apply p-2 bg-indigo-500 hover:bg-indigo-600 text-white transition
        duration-500
        ease-in-out
        transform
        hover:-translate-y-1 hover:scale-110;
	border-radius: 50%;
	font-size: 22px;
	line-height: 22px;
}

.fade-enter-active {
	animation: coming 0.4s;
	animation-delay: 0.2s;
	opacity: 0;
}

.fade-leave-active {
	animation: going 0.4s;
}

@keyframes going {
	from {
		transform: translateX(0);
	}

	to {
		transform: translateX(-10px);
		opacity: 0;
	}
}

@keyframes coming {
	from {
		transform: translateX(-10px);
		opacity: 0;
	}

	to {
		transform: translateX(0px);
		opacity: 1;
	}
}
</style>
