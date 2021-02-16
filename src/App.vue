<template>
	<div id="app">
		<div class="input-container"
			:class="[getClassOuterShadowContainer, getClassInnerShadowContainer]"
		>

			<span class="checkbox input-container__checkbox"
				:class="{ 'checkbox--active': isFocus }"
			></span>
			
			<!-- Main hidden template, get only auto height -->
			<p class="template template--hidden input-container__template"
				:class="getClassOffsetLeft"
				ref="div"
			>{{ fullValue }}</p>

			<!-- Main hidden template, leave only caret -->
			<textarea class="template template--hidden template--input input-container__template"
				:class="getClassOffsetLeft"
				:style="getStyleOffsetHeight"
				:value="fullValue"
				@input="onInput"
				@focus="isFocus = true"
				@blur="onBlur"
				@keydown.enter.prevent=""
			></textarea>

			<!-- Mirror visible template -->
			<div class="mirror template template--mirror"
				:class="getClassesMirror"
				:style="getStyleOffsetHeight"
			>
				
				<p class="before-slash">{{ mirrorValue }}</p>
				<p class="after-slash"
					v-if="isDoubleSlash"
				>
					<span class="after-slash__value"
						:style="getStyleDisplayAfterSlashValue"
					>{{ doubleSlashValue }}</span>
					<span class="after-slash__placeholder"
						v-if="isdoubleSlashPlaceholder"
					>{{ doubleSlashPlaceholder }}</span>
				</p>
			</div>

			<!-- Self icons container -->
			<transition name="icon-container">
				<div class="icon-container" v-if="isFocus">
					<div class="icon-box icon-box-left icon-container__icon-box">
						<div class="icon-box-left__framing">
							<span class="icon-box-left__line"></span>
						</div>
					</div>
					<div class="icon-box icon-box-right icon-container__icon-box">
						<div class="icon-box-right__circle"></div>
						<div class="icon-box-right__text">No list</div>
						<div class="icon-box-right__arrow"></div>
					</div>
				</div>
			</transition>

		</div>
	</div>
</template>

<script>

export default {
  	name: 'App',
	data: () => ({
		fullValue: '',
		mirrorValue: 'Write a new task',
		doubleSlashValue: '',
		doubleSlashPlaceholder: '',
		offset: 100,
		mirrorLength: 0,
		isFocus: false,
		isWritable: false,
		isDoubleSlash: false,
		isdoubleSlashPlaceholder: true,
		isAnimateDisplay: false,
		isSpace: true,
	}),
	computed: {
		getStyleOffsetHeight() {
			return { height: `${this.offset}px` };
		},
		getClassOuterShadowContainer() {
			return { 'input-container--outer-shadow': this.isFocus };
		},
		getClassInnerShadowContainer() {
			return { 'input-container--inner-shadow': this.isDoubleSlash };
		},
		getClassOffsetLeft() {
			return { 'template--focus-offset': this.isFocus };
		},
		getClassesMirror() {
			return [
				{ 'template--focus-offset': this.isFocus },
				{ 'mirror--focus-color': this.isFocus && !this.isWritable },
				{ 'mirror--writable-color': this.isFocus && this.isWritable }
			]
		},
		getStyleDisplayAfterSlashValue() {
			if (this.isAnimateDisplay) {
				return { display: 'inline' };
			}
		}
	},
	methods: {
		async onInput({ target }) {
			const convert_value = target.value.replace(/ {2,}/g, ' ');
			const double_match = convert_value.match(/\/{2}/);

			this.fullValue = convert_value;
			this.mirrorValue = convert_value;
			this.isWritable = Boolean(this.fullValue);
			

			if (double_match) {
				if (this.isSpace) {
					let modify_full_value = [...convert_value.split(''), ' '];
					modify_full_value.splice(double_match['index'], 0, ' ');

					this.fullValue = modify_full_value.join('').replace(/ {2,}/, ' ');
					this.mirrorValue = this.fullValue
						.slice(0, this.fullValue.match(/\/{2}/)['index'])
						.replace(/ /g, '\u00A0');
					this.doubleSlashValue = this.fullValue
						.slice(this.fullValue.match(/\/{2}/)['index'])
						.replace(/ /g, '\u00A0');
					this.isDoubleSlash = true;
					this.isSpace = false;
					setTimeout(() => {
						this.isAnimateDisplay = true;
					}, 600);
				} else {
					this.mirrorValue = this.fullValue.slice(0, double_match['index']);
					this.doubleSlashValue = this.fullValue.slice(double_match['index']);
					this.isdoubleSlashPlaceholder = false;
				}
			}

			await this.$nextTick();
			this.offset = this.$refs.div.offsetHeight;

			if (this.isDoubleSlash) {
				if (!this.fullValue.slice(this.fullValue.lastIndexOf('/') + 1)) {
					const full_value_of_delete = this.fullValue.slice(0, double_match['index']).trimRight();
					this.fullValue = full_value_of_delete;
					this.mirrorValue = full_value_of_delete;
					this.doubleSlashValue = '';
					this.isDoubleSlash = false;
					this.isSpace = true;
					this.isAnimateDisplay = false;
					this.isdoubleSlashPlaceholder = true;
				}
			}
		},
		onBlur() {
			this.isFocus = false;
			if(!this.isWritable) {
				this.mirrorValue = 'Write a new task';
			}
		},
	},
	watch: {
		isDoubleSlash() {
			this.doubleSlashPlaceholder = '';
			const placeholder = 'write note';
			let counter = 0;

			setTimeout(() => {
				const id = setInterval(() => {
					this.doubleSlashPlaceholder += placeholder.split('')[counter++];
					if (counter === placeholder.length) {
						clearInterval(id);
					}
				}, 30);
			}, 200);
		}
	}
}
</script>

<style lang="scss">
	@import '@/assets/scss/main';
	
	#app {
		width: 100vw;
		height: 100vh;;
		display: flex;
		align-items: center;
		justify-content: center;
		background-color: #dde2e8;
	}

	.input-container {
		width: 50vw;
		min-width: 60rem;
		height: auto;
		position: relative;
		overflow: hidden;
		border-radius: 3rem;
		background-color: rgba(79, 79, 161, .10);

		&:hover {
			background-color: rgba(79, 79, 161, .15);
		}
		&--outer-shadow {
			background-color: #fff;
			box-shadow: 0 3rem 3rem -3rem #626280;
			position: relative;

			&:hover {
				background-color: #fff;
			}
			&::after {
				content: '';
				width: 0;
				height: 3rem;
				position: absolute;
				top: 3.5rem;
				left: -.1rem;
				background-color: #dde2e8;
				clip-path: polygon(0 0, 100% 48%, 100% 52%, 0 100%);
				animation: checkbox-shadow .7s;

				@keyframes checkbox-shadow {
					10%, 30% { width: .8rem; }
				}
			}
		}
		&--inner-shadow {
			transition: .2s;
			box-shadow: inset 0 -5rem 5rem -6rem rgba(255, 165, 0, .3);
		}

		&__checkbox {
			position: absolute;
			top: 4.5rem;
			left: -1.5rem;
		}
		&__template {
			position: absolute;
			top:0;
			left: 0;
		}

	}
	.checkbox {
		width: 1.5rem;
		height: 1.5rem;
		border-radius: 1.1rem;
		background-color: #dde2e8;

		&--active {
			animation: show-checkbox .2s ease-in-out forwards;

			@keyframes show-checkbox {
				100% { top: 3.25rem; left: 2.5rem; width: 3.5rem; height: 3.5rem; }
			}

			&::after {
				content: '';
				width: 1rem;
				height: 2.5rem;
				position: absolute;
				top: 50%;
				left: -.5rem;
				transform: translateY(-50%);
				background-color: #dde2e8;
				clip-path: polygon(0 48%, 100% 5%, 100% 95%, 0 52%);
				animation: checkbox-active .2s .15s ease forwards;

				@keyframes checkbox-active {
					100% { left: 0; width: 0; }
				}
			}
		}
	}
	.template {
		width: 100%;
		min-height: 10rem;
		padding: 2rem 28rem 2rem 3.5rem;
		font-family: 'Roboto';
		font-weight: 300;
		word-break: break-all;
		line-height: 2;
		transition: .2s ease;

		&--hidden {
			background-color: transparent;
			color: transparent;
			border: none;
		}
		&--input {
			resize: none;
			outline: none;
			overflow: hidden;
			caret-color: black;
		}
		&--focus-offset {
			padding-left: 8rem;
		}
	}
	.mirror {
		color: #696868;

		&--focus-color {
			color: #999;
		}
		&--writable-color {
			color: #000;
		}
	}
	.before-slash, .after-slash {
		display: inline;
	}
	.after-slash {
		&__value {
			display: inline-block;
			transition: color .2s .6s;
			color: #8a7302;
			animation: scale-slash .5s ease forwards;

			@keyframes scale-slash {
				50% { transform: scale(1.4); }
				100% { transform: scale(1); }
			}

			&::after {
				content: '';
				width: 3rem;
				height: 3rem;
				position: absolute;
				top: 50%;
				left: 50%;
				opacity: .2;
				transform: translate(-50%, -50%);
				background: radial-gradient(circle, orange 40%, transparent);
				border-radius: 50%;
				box-shadow: 0 0 3rem orange;

				animation: slash-shadow 1.6s ease forwards;

				@keyframes slash-shadow {
					50% { width: 9rem; height: 9rem; }
					75% { opacity: 0; }
					100% { width: 0; height: 0; opacity: 0; }
				}
			}
		}
		&__placeholder {
			color: #524502;
			opacity: .4;
		}
	}
	.icon-container {
		height: 6.5rem;
		position: absolute;
		top: 1.75rem;
		right: 1.5rem;
		display: flex;
		justify-content: space-between;

		&__icon-box {
			display: flex;
			align-items: center;
		}
	}
	.icon-container-enter-active {
		opacity: 0;
		animation: icon-container-enter .2s;
		
		@keyframes icon-container-enter {
			100% { opacity: 1; }
		}
	}
	.icon-box {
		height: 100%;
		background: #ecf1f7;
		border-radius: 2rem;
		display: flex;
		align-items: center;
	}
	.icon-box-left {
		width: 6.5rem;
		margin-right: 1.5rem;		
		justify-content: center;

		&__framing {
			width: 48%;
			height: 48%;
			position: relative;
			border: .4rem solid #525c6b;
			border-radius: .9rem;

			&::after, &::before {
				content: '';
				width: .5rem;
				height: .6rem;
				border-radius: 50%;
				background-color: #525c6b;
				position: absolute;
				top: -.7rem;
			}
			&::before {
				transform: rotate(-25deg);
				left: .3rem;
			}
			&::after {
				transform: rotate(25deg);
				right: .3rem;
			}
		}
		&__line {
			width: 100%;
			height: .4rem;
			background-color: #525c6b;
			position: absolute;
			top: .6rem;
			left: 0;
		}
	}
	.icon-box-right {
		width: 17rem;
		justify-content: space-evenly;

		&__circle {
			width: 2.2rem;
			height: 2.2rem;
			border: .4rem solid #bbc0c9;
			border-radius: .8rem;
		}
		&__text {
			font-size: 2.6rem;
			color: #525c6b;
			font-weight: 600;
		}
		&__arrow {
			width: 2rem;
			height: 2rem;
			position: relative;
			
			&::before, &::after {
				content: '';
				width: 60%;
				height: .4rem;
				border-radius: .4rem;
				background: #525c6b;
				position: absolute;
				top: 50%;
			}
			&::before {
				left: .1rem;
				transform: translateY(-50%) rotate(45deg);
			}
			&::after {
				right: .1rem;
				transform: translateY(-50%) rotate(-45deg);
			}
		}
	}
</style>
