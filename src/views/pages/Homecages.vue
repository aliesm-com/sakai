<script setup>
import { FilterMatchMode, FilterOperator } from '@primevue/core/api';
import { onBeforeMount, reactive, ref } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const devices = ref(null);
const filters1 = ref(null);
const loading1 = ref(null);
const expandedRows = ref([]);

// Sample network device data based on the image
const networkDevices = reactive([
    {
        id: 1,
        hostname: 'LETH02C001',
        macAddress: 'e4:5f:01:6e:bf:9a',
        ipAddress: '192.168.0.114',
        softwareVersion: 'f6cb1b2',
        location: 'LETH'
    },
    {
        id: 2,
        hostname: 'LETH02C002',
        macAddress: 'dc:a6:32:dc:b0:21',
        ipAddress: '192.168.0.106',
        softwareVersion: '2b83544',
        location: 'LETH'
    },
    {
        id: 3,
        hostname: 'LETH02C003',
        macAddress: 'dc:a6:32:e6:dc:f4',
        ipAddress: '192.168.0.107',
        softwareVersion: 'f6cb1b2',
        location: 'LETH'
    },
    {
        id: 4,
        hostname: 'LETH02C004',
        macAddress: 'dc:a6:32:ca:25:df',
        ipAddress: '192.168.0.108',
        softwareVersion: 'f6cb1b2',
        location: 'LETH'
    },
    {
        id: 5,
        hostname: 'LETH02C005',
        macAddress: 'dc:a6:32:ca:1b:a1',
        ipAddress: '192.168.0.110',
        softwareVersion: 'f6cb1b2',
        location: 'LETH'
    }
]);

function editDevice(device) {
    // Navigate to individual homecage page
    router.push({ name: 'homecage', params: { id: device.hostname } });
}

function deleteDevice(device) {
    // Delete functionality for devices
    console.log('Delete device:', device);
}

function viewDevice(device) {
    // Navigate to individual homecage page
    router.push({ name: 'homecage', params: { id: device.hostname } });
}

onBeforeMount(() => {
    devices.value = networkDevices;
    loading1.value = false;
    initFilters1();
});

function initFilters1() {
    filters1.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        hostname: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
        macAddress: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.CONTAINS }] },
        ipAddress: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.CONTAINS }] },
        softwareVersion: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.CONTAINS }] },
        location: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] }
    };
}

function clearFilter() {
    initFilters1();
}
</script>

<template>
    <div class="card">
        <div class="font-semibold text-xl mb-4">Network Devices</div>
        <DataTable
            :value="devices"
            :paginator="true"
            :rows="10"
            dataKey="id"
            :rowHover="true"
            v-model:filters="filters1"
            filterDisplay="menu"
            :loading="loading1"
            :filters="filters1"
            :globalFilterFields="['hostname', 'macAddress', 'ipAddress', 'softwareVersion', 'location']"
            showGridlines
        >
            <template #header>
                <div class="flex justify-between">
                    <Button type="button" icon="pi pi-filter-slash" label="Clear" outlined @click="clearFilter()" />
                    <IconField>
                        <InputIcon>
                            <i class="pi pi-search" />
                        </InputIcon>
                        <InputText v-model="filters1['global'].value" placeholder="Keyword Search" />
                    </IconField>
                </div>
            </template>
            <template #empty> No devices found. </template>
            <template #loading> Loading device data. Please wait. </template>
            
            <Column field="hostname" header="Hostname" style="min-width: 12rem">
                <template #body="{ data }">
                    <span 
                        class="text-blue-600 hover:text-blue-800 cursor-pointer font-semibold"
                        @click="viewDevice(data)"
                    >
                        {{ data.hostname }}
                    </span>
                </template>
                <template #filter="{ filterModel }">
                    <InputText v-model="filterModel.value" type="text" placeholder="Search by hostname" />
                </template>
            </Column>
            
            <Column field="macAddress" header="MAC Address" style="min-width: 14rem">
                <template #body="{ data }">
                    <span class="font-mono">{{ data.macAddress }}</span>
                </template>
                <template #filter="{ filterModel }">
                    <InputText v-model="filterModel.value" type="text" placeholder="Search by MAC address" />
                </template>
            </Column>
            
            <Column field="ipAddress" header="IP Address" style="min-width: 12rem">
                <template #body="{ data }">
                    <span class="font-mono">{{ data.ipAddress }}</span>
                </template>
                <template #filter="{ filterModel }">
                    <InputText v-model="filterModel.value" type="text" placeholder="Search by IP address" />
                </template>
            </Column>
            
            <Column field="softwareVersion" header="Software Version" style="min-width: 12rem">
                <template #body="{ data }">
                    <span class="font-mono text-sm">{{ data.softwareVersion }}</span>
                </template>
                <template #filter="{ filterModel }">
                    <InputText v-model="filterModel.value" type="text" placeholder="Search by version" />
                </template>
            </Column>
            
            <Column field="location" header="Location" style="min-width: 10rem">
                <template #body="{ data }">
                    <Tag :value="data.location" severity="info" />
                </template>
                <template #filter="{ filterModel }">
                    <InputText v-model="filterModel.value" type="text" placeholder="Search by location" />
                </template>
            </Column>
            
            <Column header="Action" style="min-width: 10rem">
                <template #body="{ data }">
                    <div class="flex gap-2">
                        <Button 
                            icon="pi pi-pencil" 
                            severity="info" 
                            size="small" 
                            @click="editDevice(data)"
                            v-tooltip.top="'Edit'"
                        />
                        <Button 
                            icon="pi pi-trash" 
                            severity="danger" 
                            size="small" 
                            @click="deleteDevice(data)"
                            v-tooltip.top="'Delete'"
                        />
                    </div>
                </template>
            </Column>
        </DataTable>
    </div>
</template>

<style scoped lang="scss">
.font-mono {
    font-family: 'Courier New', Courier, monospace;
}
</style>
