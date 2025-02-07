<template>
	<v-app dark>
		<v-container fluid grid-list-md style="padding: 0; height: 100%">
			<v-layout row wrap class="layout">
				<!-- Header -->
				<v-flex xs12>
					<p class="name">{{element.name}}<br />
						<span :style="'color:'+classify(element)[2]">{{classify(element)[0]}}</span>
					</p>
				</v-flex>
				<!-- Atomic Properties -->
				<v-flex md4 sm12 xs12>
					<div class="card" id="bohr">
						<p>Atom</p>
						<div id="bohr-model-container"></div>
						<div style="clear: both"></div>
						<div class="properties">
							<div class="property" style="float:left">
								<span>{{element.atomicNumber}}</span><br />Atomic Number
							</div>
							<div class="property" style="float:right">
								<span>{{element.atomicMass}}</span><br />Atomic Mass
							</div>
							<div class="property" style="float:left">
								<span v-html="convertEC(element)"></span><br />e
								<sup>-</sup> Configuration
							</div>
							<div class="property" style="float:right">
								<span>{{element.atomicRadius || 'unknown'}}</span><br />Atomic Radius
							</div>
						</div>
					</div>
				</v-flex>
				<!-- Trends -->
				<v-flex md8 sm12 xs12>
					<div class="card" id="graph">
						<p>Trends</p>
						<div class="trendWrap">
							<TrendBox height="21vw" :active="element.atomicNumber" id="graph1" />
						</div>
					</div>
				</v-flex>
				<!-- Properties -->
				<v-flex md12>
					<div class="card" style="height: 26.5vw" id="info">
						<v-tabs v-model="active" color="accent" fixed-tabs slider-color="white">
							<v-tab :key='1'>General</v-tab>
							<v-tab :key='2'>Properties</v-tab>
							<v-tab-item :key='1'>
								<div class="content">
									<p class="description">{{description}}</p>
									<div class="generalProperties">
										<p>Appearance:
											<span>{{dataJSON[element.atomicNumber - 1].appearance || 'unknown'}}</span>
										</p>
										<p>Discovery:
											<span>{{element.yearDiscovered || 'unknown'}}, by {{dataJSON[element.atomicNumber - 1].discovered_by}}</span>
										</p>
									</div>
								</div>
							</v-tab-item>
							<v-tab-item :key='2'>
								<div class="aboutList">
									<p>
										<span>{{element.standardState || 'unknown'}}</span> <br /> Phase at STP
									</p>
									<p>
										<span>{{element.density || 'unknown'}}</span><br /> Density at STP
									</p>
									<p>
										<span>{{element.meltingPoint || 'unknown'}}</span>
										<span v-if="element.boilingPoint">K</span><br /> Melting Point
									</p>
									<p>
										<span>{{element.boilingPoint || 'unknown'}}</span>
										<span v-if="element.boilingPoint">K</span><br /> Boiling Point
									</p>
									<p>
										<span>{{dataJSON[element.atomicNumber - 1].molar_heat || 'unknown'}}</span>
										<span v-if="dataJSON[element.atomicNumber - 1].molar_heat">J/molK</span><br /> Molar Heat
									</p>
									<p>
										<span>{{element.bondingType || 'unknown'}}</span><br /> Bonding Type
									</p>
									<p>
										<span>{{element.electronegativity || 'unknown'}}</span>
										<span v-if="element.electronegativity">χr</span><br /> Electronegativity
									</p>
									<p>
										<span>{{element.electronAffinity|| 'unknown'}}</span>
										<span v-if="element.electronAffinity">kJ/mol</span> <br /> Electron Affinity
									</p>
									<p>
										<span>{{element.ionizationEnergy || 'unknown'}}</span>
										<span v-if="element.ionizationEnergy">kJ/mol</span><br /> Ionization Energy
									</p>
									<p>
										<span>{{element.vanDelWaalsRadius || 'unknown'}}
											<span v-if="element.vanDelWaalsRadius">pm</span>
										</span><br /> Van Der Waals Radius
									</p>
								</div>
							</v-tab-item>
						</v-tabs>
					</div>
				</v-flex>
			</v-layout>
		</v-container>
		<!-- Footer -->
		<div class="footerWrap">
			<Footer width="88%" />
		</div>
	</v-app>
</template>

<script>
import Elements from '@/elements';
import TrendBox from './home/TrendBox';
import Footer from './Footer';
import 'atomic-bohr-model/dist/atomicBohrModel.min.js';
import General from '@/assets/elementGeneral.json';

export default {
	name: 'ElementPage',
	components: {
		TrendBox,
		Footer,
	},
	mounted() {
		//configuration for atomic model
		var atomicConfig = {
			containerId: '#bohr-model-container',
			numElectrons: this.element.atomicNumber,
			nucleusColor: this.classify(this.element)[3],
			electronRadius: 2.5,
			electronColor: this.classify(this.element)[2],
			orbitalWidth: 1,
			orbitalColor: this.classify(this.element)[3],
			idNumber: 10,
			animationTime: 1400,
			orbitalRotationConfig: {
				pattern: {
					alternating: false,
					clockwise: false,
					preset: 'cubedNegative',
				},
			},
			symbolOffset: 8,
			drawSymbol: true,
		};
		var myAtom = new Atom(atomicConfig);
	},
	//determine which element to display
	created: function() {
		var id = parseInt(this.$route.params.id);
		this.element = Elements.find(x => x.atomicNumber === id);
		this.description = General[this.element.atomicNumber - 1].summary;
	},
	data() {
		return {
			active: null,
			description: '',
			dataJSON: General,
			element: null,
			nonMetal: [1, 6, 7, 8, 15, 16, 34],
			alkali: [3, 11, 19, 37, 55, 87],
			akaliEarth: [4, 12, 20, 38, 56, 88],
			// prettier-ignore
			transitionMetal: [21, 22,23, 24, 25, 26, 27, 28, 29, 30, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 72, 73, 74, 75, 76, 77, 78, 79, 80, 104, 105, 106, 107, 108, 109, 110, 111, 112],
			postTransition: [13, 31, 49, 50, 81, 82, 83, 113, 114, 115, 116],
			metalloid: [5, 14, 32, 33, 51, 52, 84],
			halogen: [9, 17, 35, 53, 85, 117],
			noble: [2, 10, 18, 36, 54, 86, 118],
			lanthanoid: [57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71],
			actinoid: [89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103],
		};
	},
	methods: {
		//determine element styles
		classify(element) {
			var n = element.atomicNumber;
			if (this.nonMetal.includes(n)) {
				if ([7, 8].indexOf(n) > -1) {
					return ['Diatomic Nonmetal', 'nonMetal', 'rgba(91, 93, 153, 0.9)', 'rgba(51, 53, 113, 1)'];
				} else {
					return ['Polyatomic Nonmetal', 'nonMetal', 'rgba(86, 88, 148, 0.9)', 'rgba(56, 58, 118, 1)'];
				}
			} else if (this.alkali.includes(n)) {
				return ['Alkali Metal', 'alkali', 'rgba(120, 80, 90, 0.9)', 'rgba(85, 45, 55, 1)'];
			} else if (this.akaliEarth.includes(n)) {
				return ['Alkali Earth Metal', 'alkaliEarth', 'rgba(133, 113, 101, 0.9)', 'rgba(83, 63, 51, 1)'];
			} else if (this.transitionMetal.includes(n)) {
				return ['Transition Metal', 'transitionMetal', 'rgba(99, 113, 138, 0.9)', 'rgba(54, 68, 93, 1)'];
			} else if (this.postTransition.includes(n)) {
				return ['Post Transition Metal', 'postTransition', 'rgba(74, 134, 119, 0.9)', 'rgba(34, 84, 79, 1)'];
			} else if (this.halogen.includes(n)) {
				return ['Halogen', 'halogen', 'rgba(122, 120, 181, 0.9)', 'rgba(57, 55, 106, 1)'];
			} else if (this.noble.includes(n)) {
				return ['Noble Gas', 'noble', 'rgba(136, 100, 170, 0.9)', 'rgba(76, 40, 110, 1)'];
			} else if (this.lanthanoid.includes(n)) {
				return ['Lanthanoid', 'lanthanoid', 'rgba(120, 107, 151, 0.9)', 'rgba(70, 57, 101, 1)'];
			} else if (this.actinoid.includes(n)) {
				return ['Actinoid', 'actinoid', 'rgba(110, 89, 121, 0.9)', 'rgba(62, 41, 73, 1)'];
			} else if (this.metalloid.includes(n)) {
				return ['Metalloid', 'metalloid', 'rgba(74, 114, 146, 0.9)', 'rgba(27, 67, 99, 1)'];
			}
		},
		//convert electron configuration to readable form
		convertEC(element) {
			var ec = element.electronicConfiguration.split('');
			var en = element.atomicNumber;
			for (var i = 0; i < ec.length; i++) {
				if (ec[i].match(/[a-z]/i) && i > 3) {
					ec[i + 1] = '<sup style="font-size: 10px">' + ec[i + 1] + '</sup>';
					if (ec[i + 2] && ec[i + 2] !== ' ') {
						ec[i + 2] = '<sup style="font-size: 10px">' + ec[i + 2] + '</sup>';
						i++;
					}
					i++;
				}
			}
			if (parseInt(element.atomicNumber) > 2) {
				ec.splice(0, 0, '<span style="color: rgba(255, 255, 255, 0.5)">');
				ec.splice(5, 0, '</span>');
			}
			return ec.join('');
		},
	},
};
</script>

<style lang="scss" scoped>
.application.theme--dark {
	background: rgb(26, 31, 44) !important;
	.layout {
		width: 90%;
		margin: auto !important;
		.name {
			font-size: 40px;
			font-weight: 300;
			text-align: center;
			padding: 30px;
			line-height: 40px;
			width: 100%;
			max-height: 100px;
			span {
				font-size: 25px;
			}
		}
		.card {
			text-align: center;
			background: #282e3c;
			height: 32vw;
			margin: 0.8vw;
			position: relative;
			p {
				font-size: 27px;
				font-weight: 300;
				padding: 20px;
			}
			#bohr-model-container {
				width: 90%;
				margin: -40px auto 0 auto;
				height: 18vw;
				text {
					font-family: 'Open Sans', sans-serif;
				}
			}
			.trendWrap {
				position: absolute;
				width: 100%;
				bottom: 0.5vw;
			}
			.content {
				padding: 1.5vw;
				height: 21.5vw;
				box-sizing: border-box;
				position: relative;
				.description {
					font-size: 18px;
					font-weight: 300;
					opacity: 0.8;
					text-align: left;
				}
				.generalProperties {
					position: absolute;
					bottom: 0;
					text-align: left;
					p {
						font-size: 18px;
						padding-top: 0;
						padding-bottom: 0;
						opacity: 1;
						span {
							opacity: 0.5;
						}
					}
				}
			}
			.aboutList {
				width: 95%;
				margin: auto;
				p {
					margin-top: 2vw;
					text-align: center;
					font-weight: 300;
					font-size: 1.3vw;
					margin-bottom: 0;
					float: left;
					width: 20%;
					span {
						font-size: 1.8vw;
						opacity: 0.7;
					}
				}
			}
			.properties {
				position: absolute;
				width: 100%;
				bottom: 1vw;
				sup {
					margin-left: -3px;
					font-size: 10px !important;
				}
				.property {
					margin-bottom: 15px;
					width: 50%;
					font-weight: 300;
					opacity: 0.9;
					font-size: 13px;
					span {
						font-size: 15px;
						font-weight: 300;
						opacity: 0.6;
					}
					sup {
						margin-left: -3px;
					}
				}
			}
		}
	}
	.footerWrap {
		width: 100%;
		margin-top: 2vw;
	}
}
@media only screen and (max-width: 960px) {
	.application.theme--dark {
		background: rgba(30, 36, 50, 1);
		.layout {
			width: 90%;
			margin: auto;
			.name {
				font-size: 40px;
				font-weight: 300;
				text-align: center;
				padding: 30px;
				line-height: 40px;
				width: 100%;
				max-height: 100px;
				span {
					font-size: 25px;
				}
			}
			.card {
				text-align: center;
				background: #282e3c;
				height: 32vw;
				margin: 0.8vw;
				position: relative;
				p {
					font-size: 27px;
					font-weight: 300;
					padding: 20px;
				}
				#bohr-model-container {
					width: 80%;
					margin: -70px auto 0 auto;
					height: 58vw;
					text {
						font-family: 'Open Sans', sans-serif;
					}
				}
				.trendWrap {
					position: absolute;
					width: 100%;
					bottom: 0.5vw;
				}
				.content {
					padding: 1.5vw;
					height: 38vw;
					box-sizing: border-box;
					position: relative;
					.description {
						font-size: 18px;
						font-weight: 300;
						opacity: 0.8;
						text-align: left;
					}
					.generalProperties {
						position: absolute;
						bottom: 0;
						text-align: left;
						p {
							font-size: 18px;
							padding-top: 0;
							padding-bottom: 0;
							opacity: 1;
							span {
								opacity: 0.5;
							}
						}
					}
				}
				.aboutList {
					width: 90%;
					margin: auto;
					p {
						margin-top: 2vw;
						text-align: center;
						font-weight: 300;
						font-size: 1.3vw;
						margin-bottom: 0;
						float: left;
						width: 20%;
						span {
							font-size: 1.8vw;
							opacity: 0.7;
						}
					}
				}
				.properties {
					position: absolute;
					width: 100%;
					bottom: 3vw;
					sup {
						margin-left: -3px;
						font-size: 10px !important;
					}
					.property {
						margin-bottom: 15px;
						width: 50%;
						font-weight: 300;
						opacity: 0.9;
						font-size: 20px;
						span {
							font-size: 28px;
							font-weight: 300;
							opacity: 0.6;
						}
						sup {
							margin-left: -3px;
						}
					}
				}
			}
			#bohr {
				height: 75vw;
			}
			#graph {
				height: 35vw;
			}
			#info {
				height: 45vw !important;
			}
		}
		.footerWrap {
			width: 100%;
			margin-top: 2vw;
		}
	}
}
@media only screen and (max-width: 800px) {
	.application.theme--dark {
		background: rgba(30, 36, 50, 1);
		.layout {
			width: 90%;
			margin: auto;
			.name {
				margin-bottom: 40px;
			}
			.card {
				.content {
					padding: 1.5vw;
					height: 53vw;
					box-sizing: border-box;
					position: relative;
					.description {
						font-size: 18px;
						font-weight: 300;
						opacity: 0.8;
						text-align: left;
					}
					.generalProperties {
						position: absolute;
						bottom: 0;
						text-align: left;
						p {
							font-size: 18px;
							padding-top: 0;
							padding-bottom: 0;
							opacity: 1;
							span {
								opacity: 0.5;
							}
						}
					}
				}
				.aboutList {
					width: 100%;
					margin: auto;
					p {
						margin-top: 0vw;
						text-align: center;
						font-weight: 300;
						font-size: 12px;
						margin-bottom: 0;
						float: left;
						width: 33.3%;
						span {
							font-size: 15px;
							opacity: 0.7;
						}
					}
				}
			}
			#bohr {
				height: 85vw;
			}
			#graph {
				height: 40vw;
			}
			#info {
				height: 60vw !important;
			}
		}
	}
}
@media only screen and (max-width: 600px) {
	.application.theme--dark {
		background: rgba(30, 36, 50, 1);
		.layout {
			width: 90%;
			margin: auto;
			.name {
				margin-bottom: 40px;
			}
			.card {
				.content {
					padding: 1.5vw;
					height: 75vw;
					box-sizing: border-box;
					position: relative;
					.description {
						font-size: 18px;
						font-weight: 300;
						opacity: 0.8;
						text-align: left;
					}
					.generalProperties {
						position: absolute;
						bottom: 0;
						text-align: left;
						p {
							font-size: 18px;
							padding-top: 0;
							padding-bottom: 0;
							opacity: 1;
							span {
								opacity: 0.5;
							}
						}
					}
				}
				.aboutList {
					width: 95%;
					margin: auto;
					p {
						margin-top: 1vw;
						text-align: center;
						font-weight: 300;
						font-size: 15px;
						margin-bottom: 0;
						float: left;
						width: 33.3%;
						span {
							font-size: 19px;
							opacity: 0.7;
						}
					}
				}
			}
			#bohr {
				height: 85vw;
			}
			#graph {
				height: 40vw;
			}
			#info {
				height: 85vw !important;
			}
		}
	}
}
@media only screen and (max-width: 560px) {
	.application.theme--dark {
		background: rgba(30, 36, 50, 1);
		.layout {
			width: 90%;
			margin: auto;
			.name {
				margin-bottom: 40px;
			}
			.card {
				.content {
					padding: 1.5vw;
					height: 115vw;
					box-sizing: border-box;
					position: relative;
					.description {
						font-size: 18px;
						font-weight: 300;
						opacity: 0.8;
						text-align: left;
					}
					.generalProperties {
						position: absolute;
						bottom: 0;
						text-align: left;
						p {
							font-size: 18px;
							padding-top: 0;
							padding-bottom: 0;
							opacity: 1;
							span {
								opacity: 0.5;
							}
						}
					}
				}
				.properties {
					position: absolute;
					width: 100%;
					bottom: 3vw;
					sup {
						margin-left: -3px;
						font-size: 10px !important;
					}
					.property {
						margin-bottom: 15px;
						width: 50%;
						font-weight: 300;
						opacity: 0.9;
						font-size: 14px;
						span {
							font-size: 22px;
							font-weight: 300;
							opacity: 0.6;
						}
						sup {
							margin-left: -3px;
						}
					}
				}
				.aboutList {
					width: 95%;
					margin: auto;
					p {
						margin-top: 1vw;
						text-align: center;
						font-weight: 300;
						font-size: 15px;
						margin-bottom: 0;
						float: left;
						width: 33.3%;
						span {
							font-size: 19px;
							opacity: 0.7;
						}
					}
				}
			}
			#bohr {
				height: 85vw;
			}
			#graph {
				height: 50vw;
			}
			#info {
				height: 125vw !important;
			}
		}
	}
}
</style>
