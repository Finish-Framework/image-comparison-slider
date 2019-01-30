<template>
	<div class="after-image-container unselectable" ref="afterContainer" v-on:touchstart="startSlide" v-on:mousedown="startSlide">
		<img class="after-image" :src="configuration.after.image" :alt="configuration.after.alt" :title="afterTitle" v-on:click.prevent unselectable="on">
		<span v-show="configuration.showLabels" class="label after-label" v-bind:class="">{{ afterLabelText }}</span>
		<div class="before-image-container unselectable" v-bind:style="styles.div.beforeContainer" v-on:click.prevent unselectable="on">
			<img v-bind:style="styles.img.beforeImage" :src="configuration.before.image" :alt="configuration.before.alt" :title="beforeTitle" v-on:click.prevent>
			<span v-show="configuration.showLabels" class="label before-label" v-bind:class="">{{ beforeLabelText }}</span>
		</div>
		<span v-bind:class="cssClasses.span.handle" v-bind:style="defaultHandleStyle">
			<span v-bind:class="cssClasses.span.handleCenter" v-bind:style="styles.span.handleCenter">
				<slot>&lt &gt</slot>
			</span>
		</span>
	</div>
		
		
</template>
<script>
	import configurable from '../base/configurable.js';
	import stylable from '../base/stylable.js';
	//TODO: add conditional titles to images use alt text and make labels conditional and stylable
	export default {
		data () {
			return {
				beforeImageWidth: 100,
				afterContainer: null,
				sliding: false,
				slidePosition: 100,
				imageOffset: null,
				edge: 0,
				configurationDefaults: {
					before: {
						alt: 'Before',
						title: 'Before',
						label: 'Before',
					},
					after: {
						alt: 'After',
						title: 'After',
						label: 'After',
					},
					slideOffset: 0.5,
					isVertical: false,
					showLabels: true,
					showTitles: true,
				},
				stylesDefaults: {
					img: {
						beforeImage: {},
					},
					div: {
						beforeContainer: {},
					},
					span: {
						handle: {},
						handleCenter: {
							position: 'absolute',
							top: '50%',
							left: '50%',
							width: '32px',
							height: '32px',
							color: 'white',
							background: 'rgba(0,0,0,.5)',
							margin: '-16px 0 0 -16px',
							"text-align": 'center',
							"border-radius": '4px',
							"line-height": '32px',
						}
					}
				},
				cssClassesDefaults: {
					span: {
						handle: ['handle'],
						handleCenter: ['handle-center'],
					}
				}
			}
		},
		mixins: [configurable, stylable],
		computed: {
			afterContainerWidth () {
				if(this.configuration.isVertical) {
					return this.afterContainer.clientHeight;
				} else {
					return this.afterContainer.clientWidth;
				}	
			},
			beforeContainerWidth () {
				if(this.configuration.isVertical) {
					return {
						height: `${this.slidePosition}px`, 
						width: '100%'
					}
				} else {
					return {
						width: `${this.slidePosition}px`, 
						height: 'inherit'
					};
				}
			},
			beforeImageStyles () {
				if(this.configuration.isVertical) {
					return {
						width: '100%', 
						height: this.beforeImageWidth + 'px',
					};
				} else {
					return {
						width: this.beforeImageWidth + 'px', 
						height: 'auto'
					};
				}
			},
			defaultHandleStyle () {
				if(this.configuration.isVertical) {
					return {
						left: 0 + 'px',
						top: this.slidePosition + 'px',
						bottom:this.slidePosition + 'px',
						width:'100%',
						height:'4px',
					};
				} else {
					return {
						left: this.slidePosition + 'px',
						top: 0 + 'px',
						bottom:0,
						width: '4px',
						height:'100%',
					};
				}
			},
			beforeTitle () {
				if(this.configuration.showTitles) {
					if(this.configuration.before.title){
						return this.configuration.before.title;
					}
					return this.configuration.before.alt;
				}
				return null;
			},
			afterTitle () {
				if(this.configuration.showTitles) {
					if(this.configuration.after.title){
						return this.configuration.after.title;
					}
					return this.configuration.after.alt;
				}
				return null;
			},
			beforeLabelText () {
				if(this.configuration.showLabels) {
					if(this.configuration.before.label) {
						return this.configuration.before.label;
					}
				}
				return null;
			},
			afterLabelText () {
				if(this.configuration.showLabels) {
					if(this.configuration.after.label) {
						return this.configuration.after.label;
					}
				}
				return null;
			}
		},
		watch: {
			// defaultHandleStyle: function() {
			// 	this.styles.span.handle = 
			// }
		},
		methods: {
			initSlidePosition() {
				this.slidePosition = this.afterContainerWidth * this.configuration.slideOffset;
			},
			resize() {
				if(this.afterContainerWidth !== null)
				{
					this.beforeImageWidth = this.afterContainerWidth;
				}
				this.imageOffset = this.afterContainerWidth * this.configuration.slideOffset;
				if(this.configuration.isVertical) {
					this.edge = this.afterContainer.getBoundingClientRect().top;
				} else {
					this.edge = this.afterContainer.getBoundingClientRect().left;
				}
			},
			startSlide (event) {
				this.sliding = true;
				this.moveSlide(event);
			},
			moveSlide (event) {
				if(this.sliding) {
					if(this.configuration.isVertical) {
						var relativePos = (event.touches ? event.touches[0].pageY : event.pageY) - this.edge;
						this.slidePosition = (relativePos < 0) ? 0 : ((relativePos > this.afterContainerWidth) ? this.afterContainerWidth : relativePos);
					} else {
						var relativePos = (event.touches ? event.touches[0].pageX : event.pageX) - this.edge;
						this.slidePosition = (relativePos < 0) ? 0 : ((relativePos > this.afterContainerWidth) ? this.afterContainerWidth : relativePos);

					}
				}
			
			},
			endSlide() {
				this.sliding = false;
			},
			addEvents() {
				document.addEventListener("touchmove", this.moveSlide);
				document.addEventListener("touchend", this.endSlide);
				document.addEventListener("mousemove", this.moveSlide);
				document.addEventListener("mouseup", this.endSlide);
				window.addEventListener("resize", this.resize);
			},
			removeEvents() {
				document.removeEventListener("touchmove", this.moveSlide);
				document.removeEventListener("touchend", this.endSlide);
				document.removeEventListener("mousemove", this.moveSlide);
				document.removeEventListener("mouseup", this.endSlide);
				window.removeEventListener("resize", this.resize);
			},
		},
		mounted() {
			this.afterContainer = this.$refs.afterContainer;
			this.resize();
			this.initSlidePosition();
			this.addEvents();
		},
		beforeDestroy () {
			this.removeEvents();
		}
		
	}
</script>
<style lang="scss">
// 2 concerens default css for out of the box styling
// ovveridable and does not bleed over to other elements

// TODO: extract all inline styles to a style.js
// stylable imports style.js only if stylable flag of useInlineStyles = true
// else style.js is used to create css classes or as a utility in tailwinds

// Normal usage it to import both vue component and generated css file in js and css files
// unpackage import uses default inline styles

// Framework usage imports as a component util admist tailwinds utils


	img {
		max-width: none;
	}
	.after-image-container {
		overflow: hidden;
		position: relative;
	}
	.after-image {
		width: 100%;
		display: block;
	}
	.before-image-container {
		position: absolute;
		top: 0;
		left: 0;
    	overflow: hidden;
	}
	.handle {
		position:absolute; 
		margin-left:-2px;
		background: rgba(0,0,0,.5);
		cursor: ew-resize;
	}
	.handle-center {
		
	}
	.label {
		user-select: none;
		position: absolute;
		font-size: 0.8em;
		top: calc(50% - 0.4em - 5px);
		padding: 10px;
		background: rgba(0,0,0,.5);
		color: white;
	}
	.before-label {
		left: 0;
	}
	.after-label {
		right: 0;
	}
	.unselectable {
	   -moz-user-select: -moz-none;
	   -khtml-user-select: none;
	   -webkit-user-select: none;

	   /*
	     Introduced in IE 10.
	     See http://ie.microsoft.com/testdrive/HTML5/msUserSelect/
	   */
	   -ms-user-select: none;
	   user-select: none;
	}
</style>