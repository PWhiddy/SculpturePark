<template>
	<div>
		<room v-if="sculptures.length > 0" v-bind:sculpturesData="sculptures"></room>
		<modal name="profile-modal" class="modal-popup" height="auto" width="500px">
			{{modalText}}
		</modal>
		<modal name="profile-modal-empty" class="modal-popup" height="auto" width="500px">
			Looks like you don't have any sculptures yet.
			Try making a <router-link to="/new">new</router-link> one, or check out the <a href="/references">references</a> to see how to get started.
		</modal>
	</div>		
</template>

<script>
import Sculpture from './Sculpture.vue';
import Room from './Room.vue';

export default {
	data: function() {
        return {
			sculptures: [],
			modalText: "¯\_(ツ)_/¯ couldn't find the profile you were looking for"
        }
	},
	components : {
		sculpture: Sculpture,
		room : Room,
		roomName: ''
	},
	computed: {
		currUser () {
			return this.$store.getters.getUser;
		}
	},
	created() {
		this.$store.commit('setInitialCameraPose', [6, 2.5, 4]);
		console.log(this.$route.params.username);

		// this.$route.params.id;
		console.log('$route.params.id');
		const username = this.$route.params.username;
		if(username) {
			this.roomName = username;
			this.$store.commit('setRouteTitle', username);

			this.$store.dispatch('getUserIdFromUsername', username).then(uid => {
				if(uid) {
					this.$store.dispatch('fetchUserSculptures', username).then(sculptures => {
						this.setSculpturesAndJoinRoom(sculptures);
						if(sculptures.length <= 5) {
							let count = sculptures.length -1;
							this.$store.commit('setInitialCameraPose', [count / 2 + count, 2.5, 4] );
						}
						if(sculptures.length == 0) {
							this.modalText = "This user doesn't have any sculptures yet"
							this.$modal.show('profile-modal');
						}
					});
				} else {
					this.showModal();
				}
			});
		} else {
			//this 
			this.roomName = this.currUser.displayName;
			this.$store.commit('setProfileBadgeCount', 0);
			this.$store.dispatch('fetchUserSculptures', this.currUser.displayName).then(sculptures => {
				this.setSculpturesAndJoinRoom(sculptures);
				if(sculptures.length == 0) {
					this.$modal.show('profile-modal-empty');
				}
			})
			console.log(this.currUserID);
			// console.log(this.$store.state.scene);
		}
	},
	methods : {
		setSculpturesAndJoinRoom(sculptures) {
			if(sculptures) {
					let temp = [];
					Object.keys(sculptures).forEach(key => {
						temp.push(sculptures[key]);
					})
					temp.reverse();
					this.sculptures = temp; //array.push isn't tracked by state, resetting is
				}
			console.log(this.sculptures);
			this.$store.commit('joinRoom', this.roomName);
		},
		showModal() {
			this.$modal.show('profile-modal');
		}
	},
	destroyed: function() {
		this.$store.commit('setRouteTitle', null);
	}
};


</script>