<template>
    <div>
        <LoginCheck />
        <div class="order-listing text-sm px-4 py-6 relative">
            <Breadcrumb />

            <!-- Back Button -->
            <div class="back-button flex justify-between items-center mb-8">
                <router-link to="/" class="inline-flex items-center text-gray-600 hover:text-black">
                    <BackButton alt="back" class="back-button-common" />
                    <span class="text-lg">Back</span>
                </router-link>
            </div>

            <Loader v-if="loading" />

            <div v-else>
                <!-- Toggle Switch -->
                <div class="flex items-center sm:ml-4 w-full sm:w-auto jsutify-center sm:justify-end mb-6">
                    <div class="relative flex items-center border border-gray-300 rounded-full bg-gray-100 w-full sm:w-56 h-8 cursor-pointer p-1"
                        @click="isInProgress = !isInProgress" @keydown.space.prevent="isInProgress = !isInProgress"
                        @keydown.enter.prevent="isInProgress = !isInProgress" tabindex="0" role="switch"
                        :aria-checked="isInProgress">

                        <div class="absolute top-0 left-0 w-1/2 h-full bg-white rounded-full shadow-md transition-transform duration-300 flex items-center justify-center"
                            :class="isInProgress ? 'translate-x-full' : 'translate-x-0'">
                        </div>

                        <div class="relative flex w-full justify-between z-10 text-sm font-medium">
                            <span class="w-1/2 text-center" :class="{ 'text-gray-500': isInProgress }">Loss</span>
                            <span class="w-1/2 text-center" :class="{ 'text-gray-500': !isInProgress }">In
                                Progress</span>
                        </div>
                    </div>
                </div>

                <!-- Department & Employee -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                    <!-- Select Department Select Field -->
                    <div :class="(isInProgress || waxTreeLossRecovery) ? 'w-full' : 'col-span-2 w-full'">
                        <label class="block text-sm font-semibold mb-1">Select Department <span class="text-red-500">*</span></label>
                        <select v-model="selectedDepartment" @change="fetchItemInventoryDetails"
                            class="pl-4 pr-4 py-2 border rounded-md w-full min-w-[200px]">
                            <option value="" disabled>Select Department</option>
                            <option v-for="department in departments" :key="department.value" :value="department.value">
                                {{ department.name }}
                            </option>
                        </select>
                    </div>

                    <!-- Loss Transactions (only in progress or waxTreeLossRecovery) -->
                    <div v-if="isInProgress || waxTreeLossRecovery" class="w-full">
                        <label class="block text-sm font-semibold mb-1">Loss Transactions</label>
                        <select v-model="selectedLossTransaction"
                            class="pl-4 pr-4 py-2 border rounded-md w-full min-w-[200px]">
                            <option value="">Select Loss Transaction</option>
                            <option v-for="transaction in lossTransactions" :key="transaction.value"
                                :value="transaction.value"> {{ transaction.text }} </option>
                        </select>
                    </div>
                    
                    <!-- Select Employee Select Field (only in progress or waxTreeLossRecovery) -->
                    <div v-if="isInProgress || waxTreeLossRecovery" class="w-full">
                        <label class="block text-sm font-semibold mb-1">Select Employee <span
                                class="text-red-500">*</span></label>
                        <select v-model="selectedEmployee" :disabled="loadingMetal || loadingTable"
                            class="pl-4 pr-4 py-2 border rounded-md w-full min-w-[200px]">
                            <option value="">Select Employee</option>
                            <option v-for="employee in regularEmployees[selectedDepartment]" :key="employee.value"
                                :value="employee.value"> {{ employee.text }} </option>
                        </select>
                    </div>


                    <!-- Employee Loss Quantity (only in progress) -->
                    <div v-if="isInProgress && selectedEmployee" class="w-full">
                        <label class="block text-sm font-semibold mb-1">Employee Loss Quantity</label>
                        <input type="text" v-model="employeeLossQuantity" disabled
                            class="pl-4 pr-4 py-2 border rounded-md w-full min-w-[200px] bg-gray-100" />
                    </div>

                    <!-- Employee Pure Loss Quantity (only in progress) -->
                    <div v-if="isInProgress && selectedEmployee" class="w-full">
                        <label class="block text-sm font-semibold mb-1">Employee Pure Loss Quantity</label>
                        <input type="text" v-model="employeePureLossQuantity" disabled
                            class="pl-4 pr-4 py-2 border rounded-md w-full min-w-[200px] bg-gray-100" />
                    </div>

                    <!-- Last Exit Time (only in progress) -->
                    <div v-if="isInProgress && selectedEmployee" class="w-full">
                        <label class="block text-sm font-semibold mb-1">Last Recovery Date</label>
                        <input type="text" v-model="lastExitTime" disabled
                            class="pl-4 pr-4 py-2 border rounded-md w-full min-w-[200px] bg-gray-100" />
                    </div>

                    <!-- Loss Last Transferred (only in progress) -->
                    <div v-if="isInProgress && selectedEmployee" class="w-full">
                        <label class="block text-sm font-semibold mb-1">Loss Last Transferred</label>
                        <input type="text" v-model="lossLastTransferred" disabled
                            class="pl-4 pr-4 py-2 border rounded-md w-full min-w-[200px] bg-gray-100" />
                    </div>
                </div>

                <div v-if="selectedDepartment && (isInProgress || waxTreeLossRecovery)" class="mb-6 mt-6 overflow-x-auto border p-4 rounded-md shadow-md bg-white">
                    <!-- Metal & Inventory Details -->
                    <div class="w-full grid grid-cols-2 gap-4 min-w-[200px] md:min-w-0" v-if="isInProgress || waxTreeLossRecovery"
                        :class="(isInProgress || waxTreeLossRecovery) ? 'grid-cols-1 sm:grid-cols-2' : 'grid-cols-1'">
                        <!-- Recovered Metal -->
                        <div :class="(isInProgress || waxTreeLossRecovery) ? 'w-full' : 'col-span-2 w-full'">
                            <div class="flex items-center">
                                <label class="block font-medium mb-1">Recovered Metal</label>
                                <span class="text-red-500">*</span>
                            </div>
                            <select v-model="selectedMetal"
                                class="pl-4 pr-4 py-2 border rounded-md w-full focus:ring-1 focus:ring-black"
                                :class="{ 'border-red-500': !selectedMetal }"
                                :disabled="!allItemInventoryDetails.length || loadingMetal">
                                <option value="" disabled>Select Metal</option>
                                <option v-for="metal in metalList" :key="metal.value" :value="metal.value">
                                    {{ metal.name }}
                                </option>
                            </select>
                        </div>

                        <!-- Inventory Name -->
                        <div :class="(isInProgress || waxTreeLossRecovery) ? 'w-full' : 'col-span-2 w-full'">
                            <div class="flex items-center">
                                <label class="block font-medium mb-1">Inventory Name</label>
                                <span class="text-red-500">*</span>
                            </div>
                            <div class="border rounded-md focus-within:ring-1 focus-within:ring-black overflow-hidden min-w-[200px]"
                                :class="{ 'border-red-500': inventoryNameError }">
                                <!-- Prefix with consistent size to prevent shrinking -->
                                <span
                                    class="bg-gray-200 text-gray-700 font-medium px-3 py-2 border-r whitespace-nowrap">
                                    {{ inventoryNamePreceding }}
                                </span>

                                <!-- Input area adjusts to the remaining space -->
                                <input type="text" v-model="inventoryName" placeholder="Enter Inventory Name..."
                                    @input="debouncedValidateName()"
                                    class="px-4 py-2 w-full focus:outline-none focus:ring-0"
                                    :disabled="!itemInventoryDetailsList.length || employeeRecoveryCards.length > 0" />
                            </div>
                        </div>
                        
                        <div class="bg-gray-50 p-3 rounded-lg border mt-4">
                            <!-- Original Total Loss & Total Pure Loss (constant) -->
                            <div v-if="isInProgress || waxTreeLossRecovery"
                                :class="(isInProgress || waxTreeLossRecovery) ? 'flex flex-col md:flex-row md:space-x-4 space-y-4 md:space-y-0' : 'col-span-2 flex flex-col md:flex-row md:space-x-4 space-y-4 md:space-y-0'">
                                <div class="flex-1">
                                    <label class="block font-medium mb-1">Total Loss</label>
                                    <input type="text" v-model="originalTotalLoss" disabled
                                        class="border rounded-md px-4 py-2 w-full bg-gray-100 focus:ring-2 focus:ring-blue-300" />
                                </div>
                                <div class="flex-1">
                                    <label class="block font-medium mb-1">Total Pure Loss</label>
                                    <input type="text" v-model="originalTotalPureLoss" disabled
                                        class="border rounded-md px-4 py-2 w-full bg-gray-100 focus:ring-2 focus:ring-blue-300" />
                                </div>
                            </div>

                            <!-- Total Loss & Total Pure Loss -->
                            <div :class="(isInProgress || waxTreeLossRecovery) ? 'mt-4 flex flex-col md:flex-row md:space-x-4 space-y-4 md:space-y-0' : 'col-span-2 flex flex-col md:flex-row md:space-x-4 space-y-4 md:space-y-0'">
                                <div class="flex-1">
                                    <label class="block font-medium mb-1">Remaining Total Loss</label>
                                    <input type="text" v-model="totalLoss" disabled
                                        class="border rounded-md px-4 py-2 w-full bg-gray-100 focus:ring-2 focus:ring-blue-300" />
                                </div>
                                <div class="flex-1">
                                    <label class="block font-medium mb-1">Remaining Total Pure Loss</label>
                                    <input type="text" v-model="totalPureLoss" disabled
                                        class="border rounded-md px-4 py-2 w-full bg-gray-100 focus:ring-2 focus:ring-blue-300" />
                                </div>
                            </div>
                        </div>

                        <div class="bg-gray-50 p-3 rounded-lg border mt-4">
                            <!-- Recovered Quantity, Recovery Weight Touch, Pure Recovered Quantity -->
                            <div v-if="isInProgress || waxTreeLossRecovery" class="w-full">
                                <div class="flex items-center">
                                    <label class="block font-medium mb-1 text-sm">Recovered Quantity</label>
                                    <span class="text-red-500">*</span>
                                </div>
                                <div class="border rounded-md focus-within:ring-1 focus-within:ring-black overflow-hidden w-1/2"
                                    :class="{ 'border-red-500': totalQuantityError }">
                                    <input type="number" v-model="totalRecoveredQty"
                                        :min="0" @input="debouncedValidateQuantity()" :step="0.0001"
                                        class="px-4 py-2 w-full focus:outline-none focus:ring-0 text-sm"
                                        :disabled="!itemInventoryDetailsList.length" @keydown="handleKeydown($event)" />
                                </div>
                            </div>
        
                            <div v-if="isInProgress || waxTreeLossRecovery" class="w-full mt-4 flex items-center gap-4">
                                <div class="flex-1">
                                    <div class="flex items-center">
                                        <label class="block font-medium mb-1 text-sm">Recovery Weight Touch</label>
                                        <span class="text-red-500">*</span>
                                    </div>
                                    <div class="border rounded-md focus-within:ring-1 focus-within:ring-black overflow-hidden">
                                        <input type="number" v-model="recoveryWeightTouch"
                                            :min="0" :max="100" @input="debouncedValidateQuantity()" :step="0.0001"
                                            class="px-4 py-2 w-full focus:outline-none focus:ring-0 text-sm"
                                            :disabled="!itemInventoryDetailsList.length" @keydown="handleKeydown($event)" />
                                    </div>
                                </div>
                                <div v-if="isInProgress || waxTreeLossRecovery" class="flex-1 self-center">
                                    <label class="block font-medium mb-1 text-sm">Pure Recovered Qty</label>
                                    <input type="text" v-model="pureRecoveredQty" disabled class="border rounded-md px-4 py-2 w-full bg-gray-100 text-sm" />
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Table Loader -->
                <div v-if="loadingTable" class="loader-container">
                    <div class="loader"></div>
                </div>

                <div v-else-if="itemInventoryDetailsList.length > 0">
                    <!-- Components Table -->
                    <div class="mb-6 border p-4 rounded-md shadow-md bg-white overflow-x-auto">
                        <h2 class="text-lg font-bold mb-4">Gold Details</h2>
                        <table class="min-w-full border-collapse bg-white shadow-md rounded-md">
                            <thead>
                                <tr class="bg-gray-100 text-left">
                                    <th v-if="!isInProgress && !waxTreeLossRecovery" class="px-4 py-2 text-center">
                                        <input type="checkbox" @change="selectAll" :checked="allSelected" />
                                    </th>
                                    <th v-if="waxTreeLossRecovery" class="px-4 py-2">Wax Tree Id</th>
                                    <th v-if="waxTreeLossRecovery" class="px-4 py-2">Wax Tree Name</th>
                                    <th class="px-4 py-2">Item Code</th>
                                    <th class="px-4 py-2">Loss Qty</th>
                                    <th class="px-4 py-2">Pure Gold Qty</th>
                                    <th v-if="waxTreeLossRecovery || isInProgress" class="px-4 py-2">Pure Gold Recovered
                                    </th>
                                    <th v-if="waxTreeLossRecovery || isInProgress" class="px-4 py-2">Recovered Qty</th>
                                    <!-- <th v-if="!waxTreeLossRecovery" class="px-4 py-2">Loss Pieces</th> -->
                                    <th v-if="!waxTreeLossRecovery" class="px-4 py-2">Inventory Details</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(component, index) in itemInventoryDetailsList" :key="index"
                                    class="border-b hover:bg-gray-50">
                                    <td v-if="!isInProgress && !waxTreeLossRecovery" class="px-4 py-2 text-center">
                                        <input type="checkbox" v-model="component.selected" />
                                    </td>
                                    <td v-if="waxTreeLossRecovery" class="px-4 py-2 break-words">
                                        {{ component.waxTreeId }}</td>
                                    <td v-if="waxTreeLossRecovery" class="px-4 py-2 break-words">
                                        {{ component.waxTreeName }}</td>
                                    <td class="px-4 py-2">{{ component.itemName }}</td>

                                    <!-- <td v-if="!isInProgress" class="px-4 py-2 text-center">{{ component.lossQty }}</td>
                                    <td v-else class="px-4 py-2 text-center">{{ component.lossOutsourcedQty }}</td> -->
                                    <td class="px-4 py-2 text-center">
                                        {{ component.lossQty || component.lossOutsourcedQty }}</td>

                                    <td class="px-4 py-2 text-center">{{ component.pureWeight }}</td>
                                    <td v-if="waxTreeLossRecovery" class="px-2 py-2 text-center">
                                        <input type="number" v-model="component.pureRecovered" :min="0"
                                            :max="component.lossQty"
                                            @input="debouncedValidateWaxTreerecoveredQty(component, 'pureRecovered')"
                                            class="px-4 py-2 w-full border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-black"
                                            @keydown="handleKeydown($event)" aria-label="Recovered Quantity" />
                                    </td>
                                    <td v-else-if="isInProgress" class="px-4 py-2 text-center">{{
                                        component.pureRecovered }}</td>

                                    <td v-if="waxTreeLossRecovery" class="px-2 py-2 text-center">
                                        <input type="number" v-model="component.recoveredQty" :min="0"
                                            :max="component.lossQty"
                                            @input="debouncedValidateWaxTreerecoveredQty(component, 'recoveredQty')"
                                            class="px-4 py-2 w-full border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-black"
                                            @keydown="handleKeydown($event)" aria-label="Recovered Quantity" />
                                    </td>
                                    <td v-else-if="isInProgress" class="px-2 py-2 text-center">
                                        {{ component.recoveredQty }} </td>

                                    <!-- <td v-if="!waxTreeLossRecovery" class="px-4 py-2 text-center">
                                        {{ component.lossPieces || component.lossOutsourcedPieces }}</td> -->

                                    <td v-if="!waxTreeLossRecovery" class="px-4 py-2">
                                        <Box class="h-6 w-6 cursor-pointer" @click="showInventoryDetails(component)" />
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <div v-if="isInProgress || waxTreeLossRecovery" class="text-right mb-5">
                        <button @click="addEmployeeCard" type="button"
                            class="bg-gray-500 text-white px-4 py-2 rounded-md shadow-sm hover:bg-gray-600  text-sm">
                            + Add Employee Loss
                        </button>
                    </div>

                    <!-- Employee Wise Recovery -->
                    <div v-if="isInProgress || waxTreeLossRecovery" class="mb-6 border p-4 rounded-md shadow-md bg-white overflow-x-auto">
                        <div class="flex items-center justify-between mb-4">
                            <h2 class="text-lg font-bold">Employee Wise Recovery</h2>
                        </div>

                        <!-- Cards -->
                        <div v-if="employeeRecoveryCards.length > 0" class="flex flex-wrap gap-3">
                            <div v-for="card in employeeRecoveryCards" :key="card.id"
                                class="border border-gray-200 rounded-lg p-3 bg-gray-50 shadow-sm w-64 flex-shrink-0 text-xs">
                                <!-- Card Header -->
                                <div class="flex items-center justify-between mb-2">
                                    <span class="font-bold text-gray-800 truncate mr-2">{{ card.employee.text }}</span>
                                    <button @click="removeEmployeeCard(card)"
                                        type="button" class="text-red-400 hover:text-red-600 flex-shrink-0">✕</button>
                                </div>
                                <!-- Fields -->
                                <div class="space-y-1 mb-2">
                                    <div class="flex justify-between">
                                        <span class="text-gray-600">Department</span>
                                        <span class="font-medium truncate ml-2 text-right">{{ card.department.text }}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span class="text-gray-600">Transaction</span>
                                        <span class="font-medium truncate ml-2 text-right">{{ card.lossTransaction.text || '—' }}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span class="text-gray-600">Rec. Qty</span>
                                        <span class="font-medium">{{ card.recoveredQty }}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span class="text-gray-600">Pure Rec. Qty</span>
                                        <span class="font-medium text-green-500">{{ card.pureRecoveredQty }}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span class="text-gray-600">Touch (%)</span>
                                        <span class="font-medium">{{ card.recoveryWeightTouch }}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span class="text-gray-600">Recovery %</span>
                                        <span class="font-medium text-blue-600">{{ card.recoveryPercentage }}%</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <p v-else class="text-gray-400 text-sm">No employee recovery entries added yet.</p>
                    </div>

                    <!-- Issue Components Button -->
                    <div class="text-right">
                        <button @click="settleLoss" :disabled="isButtonDisabled"
                            class="bg-gray-500 text-white px-6 py-2 rounded-md hover:bg-gray-600 shadow-md disabled:bg-gray-300 disabled:cursor-not-allowed">
                            {{ isInProgress || waxTreeLossRecovery ? "Recover Loss" : "Move loss" }}
                        </button>
                    </div>
                </div>
                <div v-else-if="selectedDepartment">
                    <div class="flex items-center justify-center h-40">
                        <p class="text-gray-500">No data available.</p>
                    </div>
                </div>

                <div v-if="!selectedDepartment">
                    <div class="flex items-center justify-center h-40">
                        <p class="text-gray-500">Select a department.</p>
                    </div>
                </div>

                <!-- Inventory Details Modal -->
                <transition name="fade">
                    <div v-if="showModal"
                        class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-25 backdrop-blur-sm">
                        <div class="bg-white p-4 rounded-lg shadow-lg w-[65%] max-h-fit flex flex-col modal-content">
                            <h2 class="text-md font-semibold mb-3 text-left">Inventory Details</h2>
                            <div class="w-full">
                                <!-- Scrollable Table Container -->
                                <div class="modal-table-container overflow-x-auto">
                                    <table class="w-full border-collapse bg-white shadow-md rounded-md text-sm">
                                        <thead>
                                            <tr class="bg-gray-100 text-center text-xs">
                                                <th class="px-2 py-2 break-words">Inventory Number</th>
                                                <th class="px-2 py-2 break-words">Inventory Name</th>
                                                <th class="px-2 py-2 break-words">Date Created</th>
                                                <th class="px-2 py-2 break-words">Qty</th>
                                                <th class="px-2 py-2 break-words">Pure Qty</th>
                                                <th class="px-2 py-2 break-words">Pieces</th>
                                                <th v-if="isInProgress" class="px-2 py-2 break-words">
                                                    Pure Qty Recovered/Lot</th>
                                                <th v-if="isInProgress" class="px-2 py-2 break-words">Recovered Qty/Lot
                                                </th>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr v-for="detail in selectedInventoryDetails"
                                                :key="detail?.inventoryNumber"
                                                class="border-b hover:bg-gray-50 text-xs">
                                                <td class="px-2 py-2 break-words">{{ detail?.inventoryNumber }}</td>
                                                <td class="px-2 py-2 break-words">{{ detail?.inventoryName }}</td>
                                                <td class="px-2 py-2 break-words">{{ detail?.createdDate }}</td>
                                                <td class="px-2 py-2 break-words">{{ detail?.quantityAvailable }}</td>
                                                <td class="px-2 py-2 break-words">{{ detail?.pureWeight }}</td>
                                                <td class="px-2 py-2 break-words">{{ detail?.pieces || 0 }}</td>
                                                <td v-if="isInProgress" class="px-2 py-2 break-words">
                                                    {{ detail?.pureRecoveredPerLot }}</td>
                                                <td v-if="isInProgress" class="px-2 py-2 break-words">
                                                    {{ detail?.recoveredQtyPerLot }}</td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </div>
                            </div>

                            <!-- Close Button -->
                            <div class="flex justify-end w-full mt-3">
                                <button @click="closeModal"
                                    class="bg-gray-500 text-white px-3 py-1 rounded hover:bg-gray-600">
                                    Close
                                </button>
                            </div>
                        </div>
                    </div>
                </transition>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, computed, onMounted, watch } from "vue";
import { useToast } from "vue-toast-notification";
import { useDebounceFn } from "@vueuse/core";
import Loader from "@/components/Loader.vue";
import Breadcrumb from "@/components/Breadcrumb.vue";
import LoginCheck from "@/components/NetSuiteLoginCheck.vue";
import BackButton from "@/assets/images/BackButton.vue";
import Box from "@/assets/images/Box.vue";
import { useAllRecoveryMangmtApi } from "@/composables/reports-api/recovery-mangmt-api/fetchRecoveryMangmtApi";
import dayjs from 'dayjs';
import { useUserStore } from '@/stores/userStore';

export default {
    name: "RecoveryGoldDetail",
    components: { Loader, Breadcrumb, LoginCheck, BackButton, Box },
    setup() {
        const toast = useToast();
        const userStore = useUserStore();
        const isAdmin = ref(userStore.isAdmin);
        const userId = ref(userStore.userId);
        const isInProgress = ref(false);
        const {
            loading,
            loadingTable,
            loadingMetal,
            error,
            itemInventoryDetailsList,
            fetchListItemInventoryDetails,
            successMessage,
            createInventoryAdjustment,
            lossOutsourcedItemStatusChange,
            fetchListMetalGold,
            metalList,
            listDepartments,
            fetchListDepartments,
            regularEmployees,
            getDepartmentEmployeesMap,
            employeeLossData,
            fetchEmployeeLossData,
            lossTransactions,
            fetchLossTransactions,
            createEmployeeWiseRecovery,
            fetchEmployeeWiseRecoveryByTransaction,
        } = useAllRecoveryMangmtApi();
        const showModal = ref(false);
        const selectedInventoryDetails = ref([]);
        const inventoryNameError = ref("");
        const inventoryNamePreceding = "Recovery: ";
        const inventoryName = ref("");
        const selectedMetal = ref("");
        const allItemInventoryDetails = ref([]);
        const totalLoss = ref(0);
        const totalPureLoss = ref(0);
        const existingRecoveredQty = ref(0);
        const existingPureRecoveredQty = ref(0);
        const originalTotalLoss = ref(0);
        const originalTotalPureLoss = ref(0);
        const totalRecoveredQty = ref(0);
        const totalQuantityError = ref("");
        const recoveryWeightTouch = ref(0);
        const pureRecoveredQty = computed(() => {
            const qty = parseFloat(totalRecoveredQty.value || 0);
            const touch = parseFloat(recoveryWeightTouch.value || 0);
            return Number(parseFloat(qty * (touch / 100)).toFixed(4));
        });
        const departments = ref([]);
        const selectedDepartment = ref("");
        const selectedEmployee = ref("");
        const waxTreeLossRecovery = ref(false);
        const employeeLossQuantity = ref(0);
        const employeePureLossQuantity = ref(0);
        const employeeComponentsData = ref({});
        const loadingEmployeeLossData = ref(false);
        const lastExitTime = ref("");
        const lossLastTransferred = ref("");
        const selectedLossTransaction = ref("");
        const employeeRecoveryCards = ref([]);

        // Computed Property to check if all items are selected
        const allSelected = computed({
            get: () => itemInventoryDetailsList.value.length > 0 && itemInventoryDetailsList.value.every(component => component.selected),
            set: (value) => { itemInventoryDetailsList.value.forEach(component => component.selected = value) }
        });

        const isButtonDisabled = computed(() => {
            if (isInProgress.value || waxTreeLossRecovery.value) {
                // Enabled if there's at least one card, OR if there are items with recoveredQty > 0
                const hasCards = employeeRecoveryCards.value.length > 0;
                const hasRecoveredItems = itemInventoryDetailsList.value?.filter(comp => Number(comp.recoveredQty || 0) > 0).length > 0;
                return loading.value || (!hasCards && !hasRecoveredItems);
            } else {
                return loading.value || itemInventoryDetailsList.value?.filter(comp => comp.selected).length === 0;
            }
        });


        // Select All Items
        const selectAll = (event) => {
            const isChecked = event.target.checked;
            itemInventoryDetailsList.value.forEach(component => {
                component.selected = isChecked;
            });
        };

        watch([isInProgress], async () => {
            // Clear selected department and refetch departments when tab changes
            selectedDepartment.value = "";
            await clearData();
            await fetchDepartments();
        });

        watch([selectedDepartment], async () => {
            if (selectedDepartment.value) {
                fetchLossTransactions({ department_id: selectedDepartment.value });
            } else {
                lossTransactions.value = [];
                selectedLossTransaction.value = "";
            }
        });

        watch([selectedEmployee], async () => {
            if (selectedEmployee.value && selectedDepartment.value && isInProgress.value) {
                await handleEmployeeSelection();
            } else {
                employeeLossQuantity.value = 0;
                employeePureLossQuantity.value = 0;
                employeeComponentsData.value = {};
                lastExitTime.value = "";
                lossLastTransferred.value = "";
            }
        });

        watch([selectedLossTransaction], async () => {
            if (selectedLossTransaction.value) {
                const found = lossTransactions.value.find(t => t.value === selectedLossTransaction.value);
                lossLastTransferred.value = found ? found.trandate : "";
                console.log('Selected Loss Transaction data:', JSON.parse(JSON.stringify(found || {})));

                // Fetch existing employee wise recovery records for this transaction
                const existingRecords = await fetchEmployeeWiseRecoveryByTransaction(selectedLossTransaction.value);
                console.log("Existing Employee Wise Recovery Records:", existingRecords);

                const totalRecoveredQtySum = Number(parseFloat(
                    existingRecords.reduce((s, r) => s + (parseFloat(r.recovered_qty) || 0), 0)
                ).toFixed(4));
                const totalPureRecoveredQtySum = Number(parseFloat(
                    existingRecords.reduce((s, r) => s + (parseFloat(r.pure_recovered_qty) || 0), 0)
                ).toFixed(4));
                console.log("Total Recovered Qty (existing):", totalRecoveredQtySum);
                console.log("Total Pure Recovered Qty (existing):", totalPureRecoveredQtySum);

                // Populate Gold Details table from the selected transaction's line items
                if (found && found.items && found.items.length > 0) {
                    itemInventoryDetailsList.value = found.items.map(item => ({ ...item, selected: false }));
                    allItemInventoryDetails.value = [...itemInventoryDetailsList.value];
                    totalLoss.value = Number(parseFloat(found.items.reduce((s, i) => s + (i.lossQty || 0), 0)).toFixed(4));
                    totalPureLoss.value = Number(parseFloat(found.items.reduce((s, i) => s + (i.pureWeight || 0), 0)).toFixed(4));
                    originalTotalLoss.value = totalLoss.value;
                    originalTotalPureLoss.value = totalPureLoss.value;

                    // Deduct existing records AFTER setting totals from items
                    existingRecoveredQty.value = totalRecoveredQtySum;
                    existingPureRecoveredQty.value = totalPureRecoveredQtySum;
                    totalLoss.value = Number(parseFloat(Math.max(0, totalLoss.value - totalRecoveredQtySum)).toFixed(4));
                    totalPureLoss.value = Number(parseFloat(Math.max(0, totalPureLoss.value - totalPureRecoveredQtySum)).toFixed(4));
                } else {
                    itemInventoryDetailsList.value = [];
                    allItemInventoryDetails.value = [];
                    totalLoss.value = 0;
                    totalPureLoss.value = 0;
                }
                totalRecoveredQty.value = 0;
                recoveryWeightTouch.value = 0;

                // Refetch employee loss data if employee is already selected
                if (selectedEmployee.value && selectedDepartment.value && isInProgress.value) {
                    await handleEmployeeSelection();
                }
            } else {
                lossLastTransferred.value = "";
                itemInventoryDetailsList.value = [];
                allItemInventoryDetails.value = [];
                totalLoss.value = 0;
                totalPureLoss.value = 0;
                totalRecoveredQty.value = 0;
                recoveryWeightTouch.value = 0;
                existingRecoveredQty.value = 0;
                existingPureRecoveredQty.value = 0;

                // Refetch employee loss data if employee is already selected
                if (selectedEmployee.value && selectedDepartment.value && isInProgress.value) {
                    await handleEmployeeSelection();
                }
            }
        });

        watch([selectedMetal], () => {
            // Only filter when we have a full dataset loaded — skip during clear/fetch cycles
            if (!allItemInventoryDetails.value.length) return;

            // For In Progress tab, the recovered metal is pure gold — quality filtering
            // does not apply since pure gold items have a different quality record
            // than the alloy inventory items. Show all items regardless of metal selected.
            if (isInProgress.value) return;

            const scrapQuality = metalList.value.find(m => m.value === selectedMetal.value)?.scrapQuality;
            const source = allItemInventoryDetails.value;

            const filtered = scrapQuality
                ? source.filter(item => String(item.metalQualityId ?? item.itemQuality) === String(scrapQuality))
                : [...source];

            itemInventoryDetailsList.value = filtered;

            // Recalculate totals from the filtered list
            totalLoss.value = Number(parseFloat(filtered.reduce((acc, item) => acc + (item.lossQty || item.lossOutsourcedQty || 0), 0)).toFixed(4));
            totalPureLoss.value = Number(parseFloat(filtered.reduce((acc, item) => acc + (item.pureWeight || 0), 0)).toFixed(4));
            totalRecoveredQty.value = 0;
        });

        // // Show Inventory Details
        // const showInventoryDetails = (item) => {
        //     if (isInProgress) {
        //         console.log("item.inventoryDetails", item.inventoryDetails);
        //         selectedInventoryDetails.value = item.inventoryDetails["lossQty"] || [];
        //     } else {
        //         selectedInventoryDetails.value = item.inventoryDetails["lossOutsourcedQty"] || [];
        //     }
        //     showModal.value = true;
        // };

        const showInventoryDetails = (item) => {
            const keyQty = !isInProgress.value ? "lossQty" : "lossOutsourcedQty";
            const keyPcs = !isInProgress.value ? "lossPieces" : "lossOutsourcedPieces";
            const details = item.inventoryDetails[keyQty] || [];

            // Assuming pieces have a matching inventoryNumber
            const piecesMap = (item.inventoryDetails[keyPcs] || []).reduce((map, p) => {
                map[p.inventoryNumber] = p.piecesAvailable;
                return map;
            }, {});

            selectedInventoryDetails.value = details.map(d => ({
                ...d,
                pieces: piecesMap[d.inventoryNumber] || 0
            }));

            showModal.value = true;
        };

        // Close Modal
        const closeModal = () => {
            showModal.value = false;
            selectedInventoryDetails.value = [];
        };

        // Handle Employee Selection and Fetch Loss Data
        const handleEmployeeSelection = async () => {
            try {
                loadingEmployeeLossData.value = true;
                const payload = {
                    emp_id: selectedEmployee.value,
                    department_id: selectedDepartment.value,
                    lossLastTransferred: lossLastTransferred.value,
                };
                
                await fetchEmployeeLossData(payload);
                
                if (employeeLossData.value && employeeLossData.value.status === 'SUCCESS') {
                    const data = employeeLossData.value.data;
                    employeeLossQuantity.value = data.totalLoss || 0;
                    employeePureLossQuantity.value = data.totalPureLoss || 0;
                    employeeComponentsData.value = data.components || {};
                    lastExitTime.value = data.exitTime || "";
                    lossLastTransferred.value = data.lossLastTransferred || "";
                } else {
                    employeeLossQuantity.value = 0;
                    employeePureLossQuantity.value = 0;
                    employeeComponentsData.value = {};
                    lastExitTime.value = "";
                    lossLastTransferred.value = "";
                    toast.warning("No loss data found for selected employee", { position: "top" });
                }
            } catch (error) {
                toast.error(error.message || "Failed to fetch employee loss data", { position: "top" });
                employeeLossQuantity.value = 0;
                employeePureLossQuantity.value = 0;
                employeeComponentsData.value = {};
                lastExitTime.value = "";
                lossLastTransferred.value = "";
            } finally {
                loadingEmployeeLossData.value = false;
            }
        };

        const validateTotalRecoveredQty = () => {
            if (parseFloat(totalRecoveredQty.value || 0) > parseFloat(totalLoss.value) || 0) {
                totalRecoveredQty.value = 0;
                totalQuantityError.value = "Recovered Quantity cannot exceed remaining Total Loss.";
                return false;
            }
            if (parseFloat(pureRecoveredQty.value || 0) > parseFloat(totalPureLoss.value) || 0) {
                totalRecoveredQty.value = 0;
                totalQuantityError.value = "Pure Recovered Quantity cannot exceed remaining Total Pure Loss.";
                return false;
            }
            if (parseFloat(totalRecoveredQty.value || 0) < 0) {
                totalRecoveredQty.value = 0;
                totalQuantityError.value = "Recovered Quantity cannot be negative.";
                return false;
            }
            if (totalRecoveredQty.value == 0) {
                totalQuantityError.value = "Recovered Quantity cannot be zero.";
                return false;
            }
            return true;
        };

        // Validate Quantity Input Field (Debounced)
        const debouncedValidateQuantity = useDebounceFn(() => {
            totalQuantityError.value = "";
            if (!validateTotalRecoveredQty()) {
                toast.error(totalQuantityError.value, { position: "top" });
                totalRecoveredQty.value = 0;
            }
            distributeRecoveredQtyGlobally();
        }, 500);

        const distributeRecoveredQtyGlobally = () => {
            let remainingQty = Number(pureRecoveredQty.value); // use pure recovered qty for distribution

            if (waxTreeLossRecovery.value) {
                // For wax tree data at item level
                itemInventoryDetailsList?.value?.forEach(item => {
                    if (remainingQty > 0) {
                        // Step 1: Distribute pure weight proportionally to each item
                        const pureRecoverableQty = Math.min(item.pureWeight, remainingQty);

                        // Step 2: Convert pure weight to metal quantity using purity
                        const recoverableQty = Number(parseFloat(pureRecoverableQty / item.purity).toFixed(4));

                        item.pureRecovered = Number(parseFloat(pureRecoverableQty).toFixed(4));
                        item.recoveredQty = recoverableQty;

                        remainingQty -= pureRecoverableQty;
                    } else {
                        item.recoveredQty = 0;
                        item.pureRecovered = 0;
                    }
                    // if (item.recoveredQty > 0) {
                    //     item.selected = true;
                    // } else {
                    //     item.selected = false;
                    // }
                });

            } else {
                // Calculate what percentage of totalPureLoss the user wants to recover
                const recoveryPercentage = totalPureLoss.value > 0
                    ? Number(pureRecoveredQty.value) / Number(totalPureLoss.value)
                    : 0;

                // Apply that same percentage to every lot's pureWeight uniformly
                itemInventoryDetailsList?.value?.forEach(item => {
                    let totalRecoveredForItem = 0;
                    let totalPureRecoveredForItem = 0;

                    item.inventoryDetails.lossOutsourcedQty.forEach(lot => {
                        const pureRecoveredPerLot = Number(parseFloat(lot.pureWeight * recoveryPercentage).toFixed(4));
                        const recoveredQtyPerLot = Number(parseFloat(pureRecoveredPerLot / lot.purity).toFixed(4));

                        lot.pureRecoveredPerLot = pureRecoveredPerLot;
                        lot.recoveredQtyPerLot = recoveredQtyPerLot;

                        totalPureRecoveredForItem += pureRecoveredPerLot;
                        totalRecoveredForItem += recoveredQtyPerLot;
                    });

                    // Roll up to item level (drives Gold Details table)
                    item.pureRecovered = Number(parseFloat(totalPureRecoveredForItem).toFixed(4));
                    item.recoveredQty = Number(parseFloat(totalRecoveredForItem).toFixed(4));
                });
            }
            // else {
            //     // Step 1: Gather all lots into a single array with a reference to their parent item
            //     const allLots = itemInventoryDetailsList?.value?.flatMap(item =>
            //         item.inventoryDetails.lossQty.map(lot => ({
            //             ...lot,
            //             parentItemId: item.itemId
            //         }))
            //     );

            //     // Step 2: Sort all lots by createdDate (ascending - oldest first)
            //     allLots.sort((a, b) => dayjs(a.createdDate, "DD-MMM-YYYY h:mm:ss a") - dayjs(b.createdDate, "DD-MMM-YYYY h:mm:ss a"));

            //     // Step 3: Distribute recoveredQty across all lots
            //     allLots.forEach(lot => {
            //         if (remainingQty > 0) {
            //             const recoverableQty = Math.min(lot.quantityAvailable, remainingQty);
            //             lot.recoveredQtyPerLot = Number(parseFloat(recoverableQty).toFixed(4));
            //             remainingQty -= recoverableQty;
            //         } else {
            //             lot.recoveredQtyPerLot = 0;
            //         }
            //     });

            //     // Step 4: Update each item's lots with the distributed quantities
            //     itemInventoryDetailsList?.value?.forEach(item => {
            //         let totalRecoveredForItem = 0;

            //         item.inventoryDetails.lossQty.forEach(lot => {
            //             // Find the updated lot info from the allLots array
            //             const updatedLot = allLots.find(l => l.inventoryNumber === lot.inventoryNumber && l.parentItemId === item.itemId);
            //             if (updatedLot) {
            //                 lot.recoveredQtyPerLot = Number(parseFloat(updatedLot.recoveredQtyPerLot || 0).toFixed(4));
            //                 totalRecoveredForItem += lot.recoveredQtyPerLot;
            //             }
            //         });

            //         // Add the sum of recovered quantities to the item
            //         item.recoveredQty = Number(parseFloat(totalRecoveredForItem).toFixed(4));
            //         // if (item.recoveredQty > 0) {
            //         //     item.selected = true;
            //         // } else {
            //         //     item.selected = false;
            //         // }
            //     });
            // }
        };

        const validateInventoryName = () => {
            const inventName = inventoryName.value?.trim()?.replace(/\s{2,}/g, ' ');
            // Trim only leading spaces & remove consecutive spaces within
            inventoryName.value = inventoryName.value?.replace(/^\s+/, '').replace(/\s{2,}/g, ' ');

            // Rule 1: Cannot be empty
            if (!inventName) {
                inventoryNameError.value = "Inventory Name cannot be empty.";
                return false;
            }

            // Rule 4: Length Check (3 to 90 characters)
            if (inventName.length < 3 || inventName.length > 90) {
                inventoryNameError.value = "Inventory Name must be between 3 and 90 characters.";
                return false;
            }

            // Passed all checks
            inventoryNameError.value = "";
            return true;
        };

        // Debounced Function to Reduce Validation Frequency
        const debouncedValidateName = useDebounceFn(() => {
            if (!validateInventoryName()) {
                toast.error(inventoryNameError.value, { position: "top" });
            }
        }, 500);

        const debouncedValidateWaxTreerecoveredQty = useDebounceFn((component, param) => {
            if (Number(component.pureRecovered || 0) > Number(component.pureWeight || 0)) {
                toast.error('The pure recovered quantity cannot excced the pure loss quantity for the item.', { position: "top" });
                component.recoveredQty = 0;
                component.pureRecovered = 0;
            }
            if (Number(component.recoveredQty || 0) > Number(component.lossQty || 0)) {
                toast.error('The recovered quantity cannot excced the loss quantity for the item.', { position: "top" });
                component.recoveredQty = 0;
                component.pureRecovered = 0;
            }
            if (param === 'pureRecovered' && Number(component.pureRecovered || 0) > 0) {
                component.recoveredQty = Number(parseFloat(component.pureRecovered / component.purity).toFixed(4));
            } else if (param === 'recoveredQty' && Number(component.recoveredQty || 0) > 0) {
                component.pureRecovered = Number(parseFloat(component.recoveredQty * component.purity).toFixed(4));
            } else {
                component.recoveredQty = 0;
                component.pureRecovered = 0;
            }
            // if (Number(component.recoveredQty || 0) > 0) {
            //     component.selected = true;
            // } else {
            //     component.selected = false;
            // }
        }, 300);

        const handleKeydown = (event) => {
            if (event.key === ',' || event.key === '-' || event.key === '+' || event.key === 'e' || event.key === 'E') {
                event.preventDefault(); // Prevent decimal input for serialized items
            }
        };

        // Add Employee Recovery Card
        const addEmployeeCard = () => {
            if (!selectedEmployee.value) {
                toast.error("Please select an employee.", { position: "top" });
                return;
            }
            if (!validateInventoryName()) {
                toast.error(inventoryNameError.value, { position: "top" });
                return;
            }
            if (Number(pureRecoveredQty.value || 0) <= 0) {
                toast.error("Please enter the quantity to recover.", { position: "top" });
                return;
            }

            const selectedComponents = itemInventoryDetailsList.value.filter(comp => Number(comp.recoveredQty || 0) > 0);
            const tableRecoveredQtySum = Number(parseFloat(
                selectedComponents.reduce((s, c) => s + (Number(c.recoveredQty) || 0), 0)
            ).toFixed(4));
            const employeeText = regularEmployees.value[selectedDepartment.value]
                ?.find(e => e.value === selectedEmployee.value)?.text || selectedEmployee.value;
            const metalText = metalList.value.find(m => m.value === selectedMetal.value)?.name || selectedMetal.value;
            const transactionText = lossTransactions.value.find(t => t.value === selectedLossTransaction.value)?.text || selectedLossTransaction.value;

            const existing = employeeRecoveryCards.value.find(c => c.employee.value === selectedEmployee.value);

            if (existing) {
                // Sum up numeric fields
                existing.recoveredQty = Number(parseFloat(existing.recoveredQty + tableRecoveredQtySum).toFixed(4));
                existing.recoveryWeightTouch = recoveryWeightTouch.value;
                existing.pureRecoveredQty = Number(parseFloat(existing.pureRecoveredQty + pureRecoveredQty.value).toFixed(4));
                existing.recoveryPercentage = originalTotalPureLoss.value > 0
                    ? Number(parseFloat((existing.pureRecoveredQty / originalTotalPureLoss.value) * 100).toFixed(2))
                    : 0;

                // Merge components — sum by itemName
                selectedComponents.forEach(c => {
                    const existingComp = existing.selectedComponents.find(ec => ec.itemName === c.itemName);
                    if (existingComp) {
                        existingComp.recoveredQty = Number(parseFloat(existingComp.recoveredQty + c.recoveredQty).toFixed(4));
                        existingComp.pureRecovered = Number(parseFloat(existingComp.pureRecovered + c.pureRecovered).toFixed(4));
                    } else {
                        existing.selectedComponents.push({ itemId: c.itemId, itemName: c.itemName, recoveredQty: c.recoveredQty, pureRecovered: c.pureRecovered, inventoryDetails: c.inventoryDetails });
                    }
                });
            } else {
                employeeRecoveryCards.value.push({
                    id: Date.now(),
                    employee: { value: selectedEmployee.value, text: employeeText },
                    department: { value: selectedDepartment.value, text: departments.value.find(d => d.value === selectedDepartment.value)?.name || selectedDepartment.value },
                    recoveredQty: tableRecoveredQtySum,
                    recoveryWeightTouch: recoveryWeightTouch.value,
                    pureRecoveredQty: pureRecoveredQty.value,
                    recoveryPercentage: originalTotalPureLoss.value > 0
                        ? Number(parseFloat((pureRecoveredQty.value / originalTotalPureLoss.value) * 100).toFixed(2))
                        : 0,
                    lossTransaction: { value: selectedLossTransaction.value, text: transactionText },
                    selectedComponents: selectedComponents.map(c => ({
                        itemId: c.itemId,
                        itemName: c.itemName,
                        recoveredQty: c.recoveredQty,
                        pureRecovered: c.pureRecovered,
                        inventoryDetails: c.inventoryDetails,
                    })),
                });
            }

            // Decrease running totals by what was just recovered (capture before reset)
            const pureRecoveredSnapshot = pureRecoveredQty.value;

            // Reset input fields — inventory name is intentionally kept for subsequent cards
            totalRecoveredQty.value = 0;
            recoveryWeightTouch.value = 0;
            itemInventoryDetailsList.value.forEach(comp => {
                comp.recoveredQty = 0;
                comp.pureRecovered = 0;
            });

            totalLoss.value = Number(parseFloat(Math.max(0, totalLoss.value - tableRecoveredQtySum)).toFixed(4));
            totalPureLoss.value = Number(parseFloat(Math.max(0, totalPureLoss.value - pureRecoveredSnapshot)).toFixed(4));
        };

        const removeEmployeeCard = (card) => {
            totalLoss.value = Number(parseFloat(Math.min(originalTotalLoss.value, totalLoss.value + card.recoveredQty)).toFixed(4));
            totalPureLoss.value = Number(parseFloat(Math.min(originalTotalPureLoss.value, totalPureLoss.value + card.pureRecoveredQty)).toFixed(4));
            employeeRecoveryCards.value.splice(employeeRecoveryCards.value.indexOf(card), 1);
        };

        // Settle Loss Action
        const settleLoss = async () => {
            console.log("settleLoss isInProgress", isInProgress.value);
            if (isInProgress.value || waxTreeLossRecovery.value) {
                if (employeeRecoveryCards.value.length === 0) {
                    if (!selectedMetal.value) {
                        toast.error("Please select a metal.", { position: "top" });
                        return;
                    }
                    if (!validateInventoryName()) {
                        toast.error(inventoryNameError.value, { position: "top" });
                        return;
                    }
                    if (Number(pureRecoveredQty.value || 0) <= 0) {
                        toast.error("Please enter the quantity to recover.", { position: "top" });
                        return;
                    }
                    const selectedComponents = itemInventoryDetailsList.value.filter(comp => Number(comp.recoveredQty || 0) > 0);
                    if (selectedComponents.length === 0) {
                        toast.error("Please select at least one item to recover.", { position: "top" });
                        return;
                    }
                    const totalRecoveredQtySum = selectedComponents.reduce((sum, comp) => sum + (parseFloat(comp.pureRecovered) || 0), 0);
                    if (totalRecoveredQtySum != totalRecoveredQty.value) {
                        toast.error(`The total recovered quantity entered must equal the sum of the recovered quantity entered for each item.`, { position: "top" });
                        return;
                    }
                    const invalidRecoveredQty = selectedComponents.filter(comp => parseFloat(comp.recoveredQty || 0) <= 0);
                    if (invalidRecoveredQty.length > 0) {
                        toast.error("Recovered quantity must be greater than 0 for all selected items.", { position: "top" });
                        return;
                    }
                    if (!selectedEmployee.value) {
                        toast.error("Please select an employee.", { position: "top" });
                        return;
                    }
                }

                // Build payloads for each employee card
                if (employeeRecoveryCards.value.length > 0) {
                    const invAdjPayloads = employeeRecoveryCards.value.map(card => ({
                        wax_tree_recovery: waxTreeLossRecovery.value,
                        selected_components: card.selectedComponents,
                        selected_metal: selectedMetal.value,
                        total_recovered_qty: card.pureRecoveredQty,
                        department_id: card.department.value,
                        inventory_name: inventoryNamePreceding + inventoryName.value,
                        selected_employee: card.employee.value,
                    }));

                    const empWiseRecoveryPayloads = employeeRecoveryCards.value.map(card => ({
                        employee_id: card.employee.value,
                        department_id: card.department.value,
                        loss_transaction_id: card.lossTransaction.value,
                        recovered_qty: card.recoveredQty,
                        pure_recovered_qty: card.pureRecoveredQty,
                        touch: card.recoveryWeightTouch,
                        recovery_percentage: card.recoveryPercentage,
                        original_total_pure_loss: originalTotalPureLoss.value,
                    }));
                }
                loading.value = true;
                try {
                    if (employeeRecoveryCards.value.length > 0) {
                        // Create one inventory adjustment per card and collect IDs
                        const empWiseRecoveryPayloads = [];

                        for (const card of employeeRecoveryCards.value) {
                            const invAdjPayload = {
                                wax_tree_recovery: waxTreeLossRecovery.value,
                                selected_components: card.selectedComponents,
                                selected_metal: selectedMetal.value,
                                total_recovered_qty: card.pureRecoveredQty,
                                department_id: card.department.value,
                                inventory_name: inventoryNamePreceding + inventoryName.value,
                                selected_employee: card.employee.value,
                            };

                            const invAdjResult = await createInventoryAdjustment(invAdjPayload);

                            if (error.value || !invAdjResult) {
                                toast.error(error.value || "Failed to create inventory adjustment for " + card.employee.text, { position: "top" });
                                loading.value = false;
                                return;
                            }

                            const invAdjId = Array.isArray(invAdjResult)
                                ? Number(invAdjResult[0])
                                : Number(invAdjResult);

                            empWiseRecoveryPayloads.push({
                                employee_id: card.employee.value,
                                department_id: card.department.value,
                                loss_transaction_id: card.lossTransaction.value,
                                recovered_qty: card.recoveredQty,
                                pure_recovered_qty: card.pureRecoveredQty,
                                touch: card.recoveryWeightTouch,
                                recovery_percentage: card.recoveryPercentage,
                                inventory_adjustment_id: invAdjId,
                            });
                        }

                        console.log("Employee Wise Recovery Payloads (with inv adj IDs):", empWiseRecoveryPayloads);

                        // Create custom records for each employee wise recovery
                        await createEmployeeWiseRecovery({ records: empWiseRecoveryPayloads });
                        if (error.value) {
                            toast.error(error.value, { position: "top" });
                        } else {
                            toast.success("Recovery records created successfully.", { position: "top" });
                            await fetchItemInventoryDetails();
                        }
                    } else {
                        // Fallback: single adjustment from current form state
                        const payload = {
                            wax_tree_recovery: waxTreeLossRecovery.value,
                            selected_components: itemInventoryDetailsList.value.filter(comp => Number(comp.recoveredQty || 0) > 0),
                            selected_metal: selectedMetal.value,
                            total_recovered_qty: pureRecoveredQty.value,
                            department_id: selectedDepartment.value,
                            inventory_name: inventoryNamePreceding + inventoryName.value,
                            selected_employee: selectedEmployee.value,
                        };
                        await createInventoryAdjustment(payload);
                        if (error.value) {
                            toast.error(error.value, { position: "top" });
                        } else if (successMessage.value) {
                            toast.success(successMessage.value, { position: "top" });
                            await fetchItemInventoryDetails();
                        } else {
                            toast.error("Failed to settle loss. Please try again.", { position: "top" });
                        }
                    }
                } catch (error) {
                    toast.error(error.message, { position: "top" });
                }
            } else {
                // !isInProgress case
                const selectedComponents = itemInventoryDetailsList.value.filter(comp => comp.selected);

                if (selectedComponents.length === 0) {
                    toast.error("Please select at least one item to proceed.", { position: "top" });
                    return;
                }

                loading.value = true;
                try {
                    const payload = {
                        status_change: true,  // Example flag to indicate status change
                        selected_components: selectedComponents,
                        department_id: selectedDepartment.value,
                    };
                    console.log("Status Change Payload", payload);

                    await lossOutsourcedItemStatusChange(payload); // <- Your API call to process status change

                    if (error.value) {
                        toast.error(error.value, { position: "top" });
                    } else if (successMessage.value) {
                        toast.success(successMessage.value, { position: "top" });
                        // await clearData();
                        await fetchItemInventoryDetails();
                    } else {
                        toast.error("Failed to update status. Please try again.", { position: "top" });
                    }
                } catch (error) {
                    toast.error(error.message, { position: "top" });
                }
            }
            loading.value = false;
        };

        const clearData = async () => {
            itemInventoryDetailsList.value.length = 0;
            allItemInventoryDetails.value = [];
            inventoryName.value = "";
            totalLoss.value = 0;
            totalPureLoss.value = 0;
            originalTotalLoss.value = 0;
            originalTotalPureLoss.value = 0;
            totalRecoveredQty.value = 0;
            recoveryWeightTouch.value = 0;
            totalQuantityError.value = "";
            selectedMetal.value = "";
            metalList.value = [];
            selectedEmployee.value = "";
            selectedLossTransaction.value = "";
            employeeRecoveryCards.value = [];
            existingRecoveredQty.value = 0;
            existingPureRecoveredQty.value = 0;
        };

        const fetchItemInventoryDetails = async () => {
            await clearData();

            if (!isInProgress.value) {
                // Loss tab — fetch from inventory balance as before
                await fetchListItemInventoryDetails({ isInProgress: false, department_id: selectedDepartment.value, material_type: "gold_type", params: 'recovery' });

                if (itemInventoryDetailsList.value[0]?.waxTreeInternalId) {
                    waxTreeLossRecovery.value = true;
                    totalLoss.value = Number(itemInventoryDetailsList.value.reduce((sum, item) => sum + (parseFloat(item.lossQty) || 0), 0).toFixed(4));
                    totalPureLoss.value = Number(itemInventoryDetailsList.value.reduce((sum, item) => sum + (parseFloat(item.pureWeight) || 0), 0).toFixed(4));
                } else {
                    waxTreeLossRecovery.value = false;
                    itemInventoryDetailsList.value = itemInventoryDetailsList.value?.filter(item => !item.totals) || [];
                    if (itemInventoryDetailsList.value?.length == 0 && error.value) {
                        toast.error(error.value, { position: "top" });
                    }
                    totalLoss.value = Number(parseFloat(itemInventoryDetailsList?.value?.reduce((acc, item) => acc + (!item.totals ? (item.lossQty || item.lossOutsourcedQty || 0) : 0), 0) || 0).toFixed(4));
                    totalPureLoss.value = Number(parseFloat(itemInventoryDetailsList?.value?.reduce((acc, item) => acc + (!item.totals ? (item.pureWeight || 0) : 0), 0) || 0).toFixed(4));
                }

                allItemInventoryDetails.value = [...itemInventoryDetailsList.value];
                originalTotalLoss.value = totalLoss.value;
                originalTotalPureLoss.value = totalPureLoss.value;
            }

            // In Progress tab — items are populated only when a loss transaction is selected.
            // Metal list is always fetched for both tabs.
            if (isInProgress.value || waxTreeLossRecovery.value) {
                await fetchListMetalGold({ department_id: selectedDepartment.value });
                if (metalList.value?.length == 1) {
                    selectedMetal.value = metalList.value[0].value;
                }
            }
        };

        const fetchDepartments = async () => {
            try {
                const payload = { 
                    all_department: isAdmin.value, 
                    user_id: userId.value, 
                    params: 'loss_recovery',
                    isInProgress: isInProgress.value,
                    material_type: 'gold_type'
                };
                await fetchListDepartments(payload); // Fetch departments via API
                if (listDepartments.value?.length > 0) {
                    departments.value = listDepartments.value;
                    
                    // Fetch employees for the new departments
                    getDepartmentEmployeesMap({ 
                        departments: listDepartments.value.map(dept => dept.value), 
                        all_employees: true 
                    });
                    
                    if (departments.value.length == 1) {
                        selectedDepartment.value = departments.value[0].value;
                    }
                } else {
                    toast.warning(error.value || "No departments found.", { position: "top" });
                }
            } catch (err) {
                toast.error(err.message || "Failed to fetch departments", { position: "top" });
            }
        };

        onMounted(async () => {
            await fetchDepartments();
            getDepartmentEmployeesMap({ departments: listDepartments.value.map(dept => dept.value), all_employees: true });
        });

        return {
            // loading Variables
            loading,
            loadingTable,
            loadingMetal,
            loadingEmployeeLossData,

            // Data Variables
            // departmentId,
            itemInventoryDetailsList,
            allItemInventoryDetails,
            showModal,
            selectedInventoryDetails,
            metalList,
            selectedMetal,
            inventoryNameError,
            totalLoss,
            totalPureLoss,
            originalTotalLoss,
            originalTotalPureLoss,
            existingRecoveredQty,
            existingPureRecoveredQty,
            totalRecoveredQty,
            totalQuantityError,
            recoveryWeightTouch,
            pureRecoveredQty,
            departments,
            selectedDepartment,
            isInProgress,
            regularEmployees,
            selectedEmployee,
            waxTreeLossRecovery,
            employeeLossQuantity,
            employeePureLossQuantity,
            employeeComponentsData,
            lastExitTime,
            lossLastTransferred,
            selectedLossTransaction,
            lossTransactions,
            employeeRecoveryCards,

            // Methods
            allSelected,
            selectAll,
            showInventoryDetails,
            closeModal,
            debouncedValidateQuantity,
            debouncedValidateName,
            handleKeydown,
            settleLoss,
            inventoryNamePreceding,
            inventoryName,
            fetchItemInventoryDetails,
            debouncedValidateWaxTreerecoveredQty,
            isButtonDisabled,
            handleEmployeeSelection,
            addEmployeeCard,
            removeEmployeeCard,
        };
    }
};
</script>