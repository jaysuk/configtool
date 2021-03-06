<template>
	<b-container>
		<b-card no-body>
			<template slot="header">
				{{$t('motors.axes')}}
				<!-- TODO add axis add buttons like for the extruders below -->
			</template>
			<table class="table mb-0">
				<thead>
					<th>{{$tc('mapping.drives', 1)}}</th>
					<th>{{$t('motors.direction')}}</th>
					<th v-show="board.stepperDriver==''">{{$t('mapping.driver')}}</th>
					<th v-show="board.microstepping">{{$t('motors.microstepping')}}</th>
					<th>{{$t('motors.perMM')}}</th>
					<th>{{$t('motors.maxSpeedChange')}}</th>
					<th>{{$t('motors.maxSpeed')}}</th>
					<th>{{$t('motors.acceleration')}}</th>
					<th>{{$t('motors.current')}}</th>
					<th v-show="template.firmware < 3">{{$t('motors.motorDriver')}}</th>
				</thead>
				<tbody>
					<drive-row v-for="i in [0, 1, 2]" :key="i" :index="i" :drivers="drivers"></drive-row>
				</tbody>
			</table>
		</b-card>

		<b-card no-body class="mt-3">
			<template #header>
				<span class="mt-2">{{$tc('motors.extruder', 2)}}</span>
				<b-button-group v-if="template.firmware < 3" class="float-right">
					<b-button size="sm" variant="success" :disabled="!canAddExtruder" @click="addExtruder">
						<font-awesome-icon icon="plus"></font-awesome-icon> {{$t('motors.addExtruder')}}
					</b-button>
					<b-button size="sm" variant="danger" :disabled="!canRemoveExtruder" @click="removeExtruder">
						<font-awesome-icon icon="minus"></font-awesome-icon> {{$t('motors.removeExtruder')}}
					</b-button>
				</b-button-group>
			</template>

			<table class="table mb-0">
				<thead>
					<th>{{$tc('mapping.drives', 1)}}</th>
					<th>{{$t('motors.direction')}}</th>
					<th v-show="board.stepperDriver==''">{{$t('mapping.driver')}}</th>
					<th v-show="board.microstepping">{{$t('motors.microstepping')}}</th>
					<th>{{$t('motors.perMM')}}</th>
					<th>{{$t('motors.maxSpeedChange')}}</th>
					<th>{{$t('motors.maxSpeed')}}</th>
					<th>{{$t('motors.acceleration')}}</th>
					<th>{{$t('motors.current')}}</th>
					<th v-show="template.firmware < 3">{{$t('motors.motorDriver')}}</th>
				</thead>
				<tbody>
					<drive-row v-for="i in template.drives.length - 3" :key="i" :index="i + 2" :drivers="drivers"></drive-row>
				</tbody>
			</table>
		</b-card>

		<b-card class="mt-3" :header="$t('motors.currentReduction')">
			<b-form-row>
				<b-col align-self="center">
					<b-checkbox v-model="idleUsed" v-preset.left="preset.idle.used" :title="$t('motors.currentReductionBox')">{{$t('motors.currentReductionDescription')}}</b-checkbox>
				</b-col>

				<b-col>
					<label for="idleFactor">{{$t('motors.idlePercentage')}}</label>
					<b-input-group append="%">
						<b-form-input id="idleFactor" v-model.number="idleFactor" v-preset="preset.idle.factor" :title="$t('motors.idlePercentageDescription')" :disabled="!idleUsed" min="0" max="100" type="number" step="any" required></b-form-input>
					</b-input-group>
				</b-col>

				<b-col>
					<label for="idleTimeout">{{$t('motors.idleTimeout')}}</label>
					<b-input-group append="s">
						<b-form-input id="idleTimeout" v-model.number="idleTimeout" v-preset="preset.idle.timeout" :title="$t('motors.idleTimeoutDescription')" :disabled="!idleUsed" min="0" type="number" step="any" required></b-form-input>
					</b-input-group>
				</b-col>
			</b-form-row>
		</b-card>
	</b-container>
</template>

<script>
'use strict';

import { mapState, mapGetters, mapMutations } from 'vuex'
import { mapFields } from 'vuex-map-fields'

import MotorsDriveRow from '../components/MotorsDriveRow.vue'
import ExpansionBoards from '../store/ExpansionBoards.js'

export default {
	components: {
		'drive-row': MotorsDriveRow
	},
	computed: {
		...mapState(['board', 'preset', 'template']),
		...mapGetters(['canAddExtruder', 'canRemoveExtruder']),
		...mapFields({
			board: 'board',
			expansionBoards: 'template.expansion_boards',
			idleUsed: 'template.idle.used',
			idleFactor: 'template.idle.factor',
			idleTimeout: 'template.idle.timeout'
		}),
		drivers() {
			const list = [];

			let index = 0;
			for (let i = 0; i < this.board.numDrives; i++) {
				switch (i) {
					case 0:
						list.push({ value: 0, text: '0 (X)' });
						break;
					case 1:
						list.push({ value: 1, text: '1 (Y)' });
						break;
					case 2:
						list.push({ value: 2, text: '2 (Z)' });
						break;
					default:
						list.push({ value: i, text: `${i} (E${i - 3})` });
						break;
				}
				index++;
			}

			for (let boardIndex = 0; boardIndex < this.expansionBoards.length; boardIndex++) {
				const expansionBoard = ExpansionBoards[this.expansionBoards[boardIndex]];
				for (let driver = 0; driver < expansionBoard.numDrives; driver++) {
					list.push({ value: index, text: `${index} (E${index - 3})` });
					index++;
				}
			}

			return list;
		}
	},
	methods: mapMutations(['addExtruder', 'removeExtruder'])
}
</script>
