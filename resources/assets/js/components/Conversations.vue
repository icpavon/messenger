<template>
	<div class="Conversations" v-show="show" transition="conversations">
		<header>
			<div>
				<i class="messages-icon"></i>
				<span class="conversations-title">Messages</span>
				<nav>
					<ul>
						<li><i @click="show = false" class="fa fa-close">&nbsp;</i></li>
					</ul>
				</nav>
			</div>
		</header>
		<div class="Conversations__filters">
			<nav>
				<ul>
					<li class="active filter-recent"><a @click="filterRecent"><i class="fa fa-history"></i> Recent</a></li>
					<li class="filter-favorites"><a @click="filterFavorites"><i class="fa fa-star"></i> Favorites</a></li>
					<li class="filter-archived"><a @click="filterArchived"><i class="fa fa-archive"></i> Archived</a></li>
					<li class="filter-blocked"><a @click="filterBlocked"><i class="fa fa-ban"></i> Blocked</a></li>
				</ul>
			</nav>
		</div>
		<div class="Conversations__search">
			<input v-model="searchText" @keyup="search"  placeholder="Search conversations">
		</div>
			<ul class="list">
				<li v-for="conversation in conversations" @click="toggleActive(conversation)" class="{{ isActive(conversation) ? 'active' : '' }}">
					<div v-if="conversation.sender_id === user.id">
						<img v-bind:src="conversation.recipient.avatar" alt="{{ conversation.recipient.name }}"/>
						<span class="conversation-content">
							<span class="conversation-name">{{ conversation.recipient.name }}</span><br>
							<span class="conversation-last-message">{{ lastMessage(conversation) }}</span>
						</span>
						<span v-if="conversation.messages.length > 0" class="last-message-timestamp">
						{{ conversation.messages[conversation.messages.length -1].created_at | timeago }}
						</span>
						<span v-else><br></span>
					</div>
					<div v-else>
						<img v-bind:src="conversation.sender.avatar" alt="{{ conversation.sender.name }}"/>
						<span class="conversation-content">
							<span class="conversation-name">{{ conversation.sender.name }}</span><br>
							<span class="conversation-last-message">{{ lastMessage(conversation) }}</span>
						</span>
						<span v-if="conversation.messages.length > 0" class="last-message-timestamp">
						{{ conversation.messages[conversation.messages.length -1].created_at | timeago }}
						</span>
						<span v-else><br></span>
					</div>
				</li>
			</ul>
	</div>
</template>

<script>
export default {
	props: {
		allConversations: {
			require: true,
			twoWay: true
		},
		conversations: {
			require: true,
			twoWay: true
		},
		show: {
			 type: Boolean,
			 required: true,
			 twoWay: true
		},
		user: {
			required: true,
			twoWay: true
		}
	},
	data() {
		return {
			apiToken: $('meta[name="api-token"]').attr('content'),
			searchText: '',
			currentFilter: 'recent'
		};
	},
	ready() {

	},
	computed: {
		isSelected() {
			return this.active;
		},
	},
	methods: {
		isActive(conversation) {
			return conversation.active;
		},
		toggleActive(conversation) {
			conversation.active = ! conversation.active;
			console.log(conversation.active);
			this.$dispatch('activateConversation', conversation);
			// scroll message into view
			setTimeout(function() {
				document.querySelector('.Chatbox__content').scrollTop = 10000000;
			}, 50);
		},
		lastMessage(conversation) {
			// return the last message and the name of the sender or You said...
			let lastMessageSent = conversation.messages[conversation.messages.length -1];

			// if we don't have a message, just return
			if (lastMessageSent === undefined) return '';

			// otherwise see if it's from the current user or not
			return lastMessageSent.user.id === this.user.id ?
				'You said ' + lastMessageSent.body.substr(0, 15) + '...' :
				lastMessageSent.user.name + ' said ' + lastMessageSent.body.substr(0, 15) + '...'
		},
		search() {
			// filter conversations by username
			this.conversations = this.conversations.filter(function(conversation) {
				console.log(conversation.sender.name);
				console.log(conversation.recipient.name);
				return conversation.sender.name.indexOf(this.searchText)  > -1 ||
					   conversation.recipient.name.indexOf(this.searchText) > -1;
			}.bind(this));

			if (this.searchText.length === 0) {
				this.filter();
			}
		},
		clearSearch() {
			this.searchText = '';
			this.conversations = this.allConversations;
		},
		filter() {
			switch(this.currentFilter) {
				case 'recent':
					this.filterRecent();
					break;
				case 'favorites':
					this.filterFavorites();
					break;
				case 'archived':
					this.filterArchived();
					break;
				case 'blocked':
					this.filterBlocked();
					break;
			}
		},
		filterRecent() {
			this.conversations = this.allConversations;
			this.currentFilter = 'recent';
			console.log(this.allConversations);
			this.conversations = this.conversations.filter(function(conversation) {
				return !conversation.favorite && !conversation.blocked && !conversation.archived;
			});
			// toggle .active css class
			// remove
			document.querySelector('.filter-favorites').classList.remove('active');
			document.querySelector('.filter-archived').classList.remove('active');
			document.querySelector('.filter-blocked').classList.remove('active');
			// add
			document.querySelector('.filter-recent').classList.add('active');
		},
		filterFavorites() {
			this.conversations = this.allConversations;
			this.currentFilter = 'favorites';
			this.conversations = this.conversations.filter(function(conversation) {
				return conversation.favorite && !conversation.blocked && !conversation.archived;
			});
			// toggle .active css class
			// remove
			document.querySelector('.filter-recent').classList.remove('active');
			document.querySelector('.filter-archived').classList.remove('active');
			document.querySelector('.filter-blocked').classList.remove('active');
			// add
			document.querySelector('.filter-favorites').classList.add('active');
		},
		filterArchived() {
			this.conversations = this.allConversations;
			this.currentFilter = 'archived';
			this.conversations = this.conversations.filter(function(conversation) {
				return !conversation.blocked && conversation.archived;
			});
			// toggle .active css class
			// remove
			document.querySelector('.filter-favorites').classList.remove('active');
			document.querySelector('.filter-recent').classList.remove('active');
			document.querySelector('.filter-blocked').classList.remove('active');
			// add
			document.querySelector('.filter-archived').classList.add('active');
		},
		filterBlocked() {
			this.conversations = this.allConversations;
			this.currentFilter = 'blocked';
			this.conversations = this.conversations.filter(function(conversation) {
				return conversation.blocked;
			});
			// toggle .active css class
			// remove
			document.querySelector('.filter-favorites').classList.remove('active');
			document.querySelector('.filter-archived').classList.remove('active');
			document.querySelector('.filter-recent').classList.remove('active');
			// add
			document.querySelector('.filter-blocked').classList.add('active');
		}
	}
}
</script>

<style>
	.Conversations {
		position: fixed;
		right: 0;
		top: 55px;
		z-index: 0;
		height: 100%;
		min-height: 100%;
		background: #ffffff;
		/*background: #e5e5e5;*/
		/*min-width: 25em;*/
		min-width: 250px;

		/* animated */
		-webkit-animation-duration: 0.5s;
	    animation-duration: 0.5s;
	    -webkit-animation-fill-mode: both;
	    animation-fill-mode: both;

		border-left: 1px solid rgba(154, 159, 170, 0.3);
	}
	.conversations-title {
		position: absolute;
	    top: 0.25em;
	    display: inline-block;
	    padding-left: 0.25em;
	}
	.messages-icon {
		background: url('../img/messages.svg') no-repeat;
		width: 16px;
	    height: 16px;
	    display: inline-block;
	    margin: 0.5em 0.25em 0 0.25em;
	}
	.Conversations header {
		background: #436790;
		color: #fff;
		position: relative;
		clear: both;
		padding: 0.25em 0;
	}

	.Conversations header div {
		position: relative;
		padding-left: 1em;
		font-weight: 500;
	}

	.Conversations header div nav {
		position: absolute;
		right: 0.25em;
		top: 0;
	}

	.Conversations header div nav li  {
		cursor: pointer;
	}

	.Conversations__filters {
		background: #E2E6EA;
	}
		.Conversations__filters nav ul li {
			display: inline-block;
			margin: 0.5em 0.25em;
		}
		.Conversations__filters nav ul li a {
			padding: 0.25em 1em;
			display: inline-block;
			border: solid 1px #E2E6EA;
			border-radius: 10px;
			font-size: 0.875em;
			cursor: pointer;
			transition: all 0.5s ease;
		}
		.Conversations__filters nav ul li a:hover, .Conversations__filters nav ul li.active a {
			border: solid 1px #fff;
			background: #fff;
			text-decoration: none;
		}
	.Conversations__search {
		border-bottom: solid 1px #E2E6EA;
	}
	.Conversations__search input {
		width: 100%;
		height: 35px;
		border: none;
		padding: 0.25em 0.5em;
	}
	.Conversations__search input:focus {
		outline: none;
	}
	.Conversations__body {
		overflow: hidden;
		max-height: 80%;
	}
	.Conversations ul {
		list-style: none;
		margin: 0;
		padding: 0;
		overflow: scroll;
		max-height: 85%;
	}

	.Conversations ul.list li {
		position: relative;
		padding: 0.5em 1em;
		border-bottom: solid 1px #F2F2F2;
		cursor: pointer;
		transition: all 0.35s ease;
		color: #455e77;
	}
	.Conversations ul.list li.active, .Conversations ul.list li.active:hover  {
		background: #edf0f3;

	}

	.Conversations ul.list li:hover {
		background: #edf0f3;
	}

	.Conversations ul li img {
		width: 25px;
		height: 25px;
		margin-right: 1em;
		border: solid 1px #ddd;
		float: left;
	}

	.conversation-content {

	}

	.conversation-name {
		font-weight: 600;
		font-size: 0.875em;
	}

	.Conversations ul.list li:hover span.last-message-timestamp,
	.Conversations ul.list li.active:hover span.last-message-timestamp,
	.Conversations ul.list li.active span.last-message-timestamp {
		color: #8793B8;
	}
	.conversation-last-message {
		font-size: 0.775em;
	}

	.last-message-timestamp {
		margin-right: 0.5em;
		padding-left: 1em;
		font-size: 0.775em;
		color: #BFC0C3;
		position: absolute;
		right: 0.5em;
		top: 0.5em;
	}
	@-webkit-keyframes slideInRight {
    from {
        -webkit-transform: translate3d(100%, 0, 0);
        transform: translate3d(100%, 0, 0);
        visibility: visible;
    }

    to {
        -webkit-transform: translate3d(0, 0, 0);
        transform: translate3d(0, 0, 0);
    }
}

@keyframes slideInRight {
    from {
        -webkit-transform: translate3d(100%, 0, 0);
        transform: translate3d(100%, 0, 0);
        visibility: visible;
    }

    to {
        -webkit-transform: translate3d(0, 0, 0);
        transform: translate3d(0, 0, 0);
    }
}
.conversations-enter {
	-webkit-animation-name: slideInRight;
    animation-name: slideInRight;
}

@-webkit-keyframes slideOutRight {
    from {
        -webkit-transform: translate3d(0, 0, 0);
        transform: translate3d(0, 0, 0);
    }

    to {
        visibility: hidden;
        -webkit-transform: translate3d(100%, 0, 0);
        transform: translate3d(100%, 0, 0);
    }
}

@keyframes slideOutRight {
    from {
        -webkit-transform: translate3d(0, 0, 0);
        transform: translate3d(0, 0, 0);
    }

    to {
        visibility: hidden;
        -webkit-transform: translate3d(100%, 0, 0);
        transform: translate3d(100%, 0, 0);
    }
}

.conversations-leave {
	-webkit-animation-name: slideOutRight;
    animation-name: slideOutRight;
}
</style>
