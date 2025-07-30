<script setup>
import { FilterMatchMode, FilterOperator } from '@primevue/core/api';
import { onBeforeMount, reactive, ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import { useConfirm } from 'primevue/useconfirm';
import { useToast } from 'primevue/usetoast';

const router = useRouter();
const confirm = useConfirm();
const toast = useToast();
const devices = ref(null);
const filters1 = ref(null);
const loading1 = ref(null);
const expandedRows = ref([]);
const settingsDialog = ref(false);
const selectedDevice = ref(null);
const layout = ref('table'); // 'table' or 'grid'
const layoutOptions = ref(['table', 'grid']);
const deviceSettings = ref({
    homecage: '',
    experiment: '',
    location: '',
    hostname: '',
    remoteStorage: '',
    temperatureHumidity: true,
    waterSensor: true,
    foodSensor: true,
    ammonia: true,
    daytimeLightingStart: new Date(2024, 0, 1, 7, 30), // 7:30 AM
    nighttimeLightingStart: new Date(2024, 0, 1, 17, 30), // 5:30 PM
    ledLightingLevel: 90,
    irLightingLevel: 90,
    videoStream: true,
    videoRecord: false,
    dailyRebootTime: new Date(2024, 0, 1, 22, 10) // 10:10 PM
});

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

function openSettings(device) {
    selectedDevice.value = device;
    // Load current settings for the device
    deviceSettings.value = {
        homecage: device.hostname,
        experiment: '',
        location: device.location,
        hostname: device.hostname,
        remoteStorage: '',
        temperatureHumidity: true,
        waterSensor: true,
        foodSensor: true,
        ammonia: true,
        daytimeLightingStart: new Date(2024, 0, 1, 7, 30),
        nighttimeLightingStart: new Date(2024, 0, 1, 17, 30),
        ledLightingLevel: 90,
        irLightingLevel: 90,
        videoStream: true,
        videoRecord: false,
        dailyRebootTime: new Date(2024, 0, 1, 22, 10)
    };
    settingsDialog.value = true;
}

function saveSettings() {
    // Save the settings for the selected device
    console.log('Saving settings for:', selectedDevice.value.hostname, deviceSettings.value);
    
    toast.add({ 
        severity: 'success', 
        summary: 'Settings Saved', 
        detail: `Settings for ${selectedDevice.value.hostname} have been updated successfully`, 
        life: 3000 
    });
    
    settingsDialog.value = false;
}

function cancelSettings() {
    settingsDialog.value = false;
}

function deleteDevice(event, device) {
    confirm.require({
        target: event.target,
        message: `Are you sure you want to delete ${device.hostname}?`,
        icon: 'pi pi-exclamation-triangle',
        rejectProps: {
            label: 'Cancel',
            severity: 'secondary',
            outlined: true
        },
        acceptProps: {
            label: 'Delete',
            severity: 'danger'
        },
        accept: () => {
            // Remove device from the array
            const index = networkDevices.findIndex(d => d.id === device.id);
            if (index !== -1) {
                networkDevices.splice(index, 1);
                toast.add({ 
                    severity: 'success', 
                    summary: 'Device Deleted', 
                    detail: `${device.hostname} has been deleted successfully`, 
                    life: 3000 
                });
            }
        },
        reject: () => {
            toast.add({ 
                severity: 'info', 
                summary: 'Cancelled', 
                detail: 'Delete operation cancelled', 
                life: 3000 
            });
        }
    });
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
        <div class="flex justify-between items-center mb-4">
            <div class="font-semibold text-xl">Network Devices</div>
            <SelectButton v-model="layout" :options="layoutOptions" :allowEmpty="false">
                <template #option="{ option }">
                    <i :class="[option === 'table' ? 'pi pi-table' : 'pi pi-th-large']" />
                    <span class="ml-2">{{ option === 'table' ? 'Table' : 'Grid' }}</span>
                </template>
            </SelectButton>
        </div>
        
        <!-- Table View -->
        <DataTable
            v-if="layout === 'table'"
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
            
            <Column header="Action" style="min-width: 14rem">
                <template #body="{ data }">
                    <div class="flex gap-2">
                        <Button 
                            icon="pi pi-pencil" 
                            severity="info" 
                            size="small" 
                            @click="openSettings(data)"
                            v-tooltip.top="'Edit'"
                        />
                        <Button 
                            icon="pi pi-trash" 
                            severity="danger" 
                            size="small" 
                            @click="deleteDevice($event, data)"
                            v-tooltip.top="'Delete'"
                        />
                    </div>
                </template>
            </Column>
        </DataTable>

        <!-- Grid View -->
        <div v-else-if="layout === 'grid'" class="grid-view">
            <div class="mb-4 flex justify-between">
                <Button type="button" icon="pi pi-filter-slash" label="Clear" outlined @click="clearFilter()" />
                <IconField>
                    <InputIcon>
                        <i class="pi pi-search" />
                    </InputIcon>
                    <InputText v-model="filters1['global'].value" placeholder="Keyword Search" />
                </IconField>
            </div>
            
            <div v-if="!devices || devices.length === 0" class="text-center py-8">
                <p class="text-gray-500">No devices found.</p>
            </div>
            
            <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
                <div 
                    v-for="device in devices" 
                    :key="device.id" 
                    class="device-card bg-white rounded-lg shadow-md border border-gray-200 overflow-hidden hover:shadow-lg transition-shadow duration-300"
                >
                    <!-- Device Image/Thumbnail -->
                    <div class="device-image-container relative">
                        <div class="aspect-video bg-gradient-to-br from-purple-900 to-purple-600 flex items-center justify-center">
                            <!-- Simulating night vision camera view -->
                            <div class="text-white text-center">
                                <i class="pi pi-video text-4xl mb-2 opacity-80"></i>
                                <div class="text-xs opacity-75">Live Feed</div>
                            </div>
                        </div>
                        
                        <!-- Device Name Overlay -->
                        <div class="absolute top-2 left-2 bg-black bg-opacity-70 text-white px-2 py-1 rounded text-sm font-semibold">
                            {{ device.hostname }}
                        </div>
                        
                        <!-- Timestamp -->
                        <div class="absolute bottom-2 right-2 bg-black bg-opacity-70 text-white px-2 py-1 rounded text-xs">
                            just now
                        </div>
                    </div>
                    
                    <!-- Device Info -->
                    <div class="p-4">
                        <div class="mb-3">
                            <h3 
                                class="font-semibold text-lg text-blue-600 hover:text-blue-800 cursor-pointer mb-1"
                                @click="viewDevice(device)"
                            >
                                {{ device.hostname }}
                            </h3>
                            <div class="text-sm text-gray-600 space-y-1">
                                <div class="flex items-center">
                                    <i class="pi pi-desktop text-gray-400 mr-2 text-xs"></i>
                                    <span class="font-mono">{{ device.macAddress }}</span>
                                </div>
                                <div class="flex items-center">
                                    <i class="pi pi-globe text-gray-400 mr-2 text-xs"></i>
                                    <span class="font-mono">{{ device.ipAddress }}</span>
                                </div>
                                <div class="flex items-center">
                                    <i class="pi pi-cog text-gray-400 mr-2 text-xs"></i>
                                    <span class="font-mono">{{ device.softwareVersion }}</span>
                                </div>
                                <div class="flex items-center">
                                    <i class="pi pi-map-marker text-gray-400 mr-2 text-xs"></i>
                                    <Tag :value="device.location" severity="info" class="text-xs" />
                                </div>
                            </div>
                        </div>
                        
                        <!-- Action Buttons -->
                        <div class="flex justify-between items-center pt-3 border-t border-gray-100">
                            <div class="flex gap-2">
                                <Button 
                                    icon="pi pi-pencil" 
                                    severity="info" 
                                    size="small" 
                                    @click="openSettings(device)"
                                    v-tooltip.top="'Edit'"
                                    outlined
                                />
                                <Button 
                                    icon="pi pi-trash" 
                                    severity="danger" 
                                    size="small" 
                                    @click="deleteDevice($event, device)"
                                    v-tooltip.top="'Delete'"
                                    outlined
                                />
                            </div>
                            <Button 
                                label="Detail" 
                                icon="pi pi-arrow-right" 
                                size="small"
                                @click="viewDevice(device)"
                                text
                                class="font-semibold"
                            />
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <ConfirmPopup />
    
    <!-- Settings Dialog -->
    <Dialog 
        v-model:visible="settingsDialog" 
        :style="{ width: '800px', maxHeight: '90vh' }" 
        header="Device Settings" 
        :modal="true"
        class="p-fluid settings-dialog"
        :dismissableMask="true"
    >
        <div class="settings-content"
             style="max-height: 70vh; overflow-y: auto; padding: 1rem;"
        >
        <div class="settings-content"
             style="max-height: 70vh; overflow-y: auto; padding: 1rem;"
        >
            <!-- Basic Information Section -->
            <div class="settings-section mb-6">
                <h3 class="section-title mb-4">
                    <i class="pi pi-cog mr-2"></i>
                    Basic Information
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <!-- Homecage -->
                    <div class="form-group">
                        <label for="homecage" class="form-label">Homecage</label>
                        <Dropdown 
                            id="homecage"
                            v-model="deviceSettings.homecage" 
                            :options="[{label: deviceSettings.homecage, value: deviceSettings.homecage}]"
                            optionLabel="label" 
                            optionValue="value"
                            placeholder="Select Homecage"
                            class="w-full"
                        />
                    </div>

                    <!-- New Experiment -->
                    <div class="form-group">
                        <label for="experiment" class="form-label">New Experiment</label>
                        <Dropdown 
                            id="experiment"
                            v-model="deviceSettings.experiment" 
                            :options="[]"
                            placeholder="Select Experiment"
                            class="w-full"
                        />
                    </div>

                    <!-- New Location -->
                    <div class="form-group">
                        <label for="location" class="form-label">New Location</label>
                        <InputText 
                            id="location"
                            v-model="deviceSettings.location" 
                            placeholder="Enter location"
                            class="w-full"
                        />
                    </div>

                    <!-- New Hostname -->
                    <div class="form-group">
                        <label for="hostname" class="form-label">New Hostname</label>
                        <InputText 
                            id="hostname"
                            v-model="deviceSettings.hostname" 
                            placeholder="Enter hostname"
                            class="w-full"
                        />
                    </div>

                    <!-- Remote Storage -->
                    <div class="form-group md:col-span-2">
                        <label for="remoteStorage" class="form-label">Remote Storage</label>
                        <Dropdown 
                            id="remoteStorage"
                            v-model="deviceSettings.remoteStorage" 
                            :options="[{label: '---------', value: ''}]"
                            optionLabel="label" 
                            optionValue="value"
                            placeholder="Select Remote Storage"
                            class="w-full"
                        />
                    </div>
                </div>
            </div>

            <!-- Sensor Settings Section -->
            <div class="settings-section mb-6">
                <h3 class="section-title mb-4">
                    <i class="pi pi-eye mr-2"></i>
                    Sensor Configuration
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div class="sensor-item">
                        <div class="flex align-items-center">
                            <Checkbox 
                                id="temperatureHumidity" 
                                v-model="deviceSettings.temperatureHumidity" 
                                :binary="true"
                                class="mr-3"
                            />
                            <label for="temperatureHumidity" class="sensor-label">
                                <i class="pi pi-sun mr-2 text-orange-500"></i>
                                Temperature & Humidity Sensor
                            </label>
                        </div>
                    </div>

                    <div class="sensor-item">
                        <div class="flex align-items-center">
                            <Checkbox 
                                id="waterSensor" 
                                v-model="deviceSettings.waterSensor" 
                                :binary="true"
                                class="mr-3"
                            />
                            <label for="waterSensor" class="sensor-label">
                                <i class="pi pi-tint mr-2 text-blue-500"></i>
                                Water Sensor
                            </label>
                        </div>
                    </div>

                    <div class="sensor-item">
                        <div class="flex align-items-center">
                            <Checkbox 
                                id="foodSensor" 
                                v-model="deviceSettings.foodSensor" 
                                :binary="true"
                                class="mr-3"
                            />
                            <label for="foodSensor" class="sensor-label">
                                <i class="pi pi-shopping-cart mr-2 text-green-500"></i>
                                Food Sensor
                            </label>
                        </div>
                    </div>

                    <div class="sensor-item">
                        <div class="flex align-items-center">
                            <Checkbox 
                                id="ammonia" 
                                v-model="deviceSettings.ammonia" 
                                :binary="true"
                                class="mr-3"
                            />
                            <label for="ammonia" class="sensor-label">
                                <i class="pi pi-cloud mr-2 text-purple-500"></i>
                                Ammonia Sensor
                            </label>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Lighting Settings Section -->
            <div class="settings-section mb-6">
                <h3 class="section-title mb-4">
                    <i class="pi pi-sun mr-2"></i>
                    Lighting Configuration
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Time Settings -->
                    <div class="lighting-times">
                        <div class="form-group mb-4">
                            <label for="daytimeLighting" class="form-label">Daytime Lighting Start</label>
                            <Calendar 
                                id="daytimeLighting"
                                v-model="deviceSettings.daytimeLightingStart" 
                                timeOnly 
                                hourFormat="12"
                                placeholder="Select time"
                                class="w-full"
                                showIcon
                            />
                        </div>

                        <div class="form-group">
                            <label for="nighttimeLighting" class="form-label">Nighttime Lighting Start</label>
                            <Calendar 
                                id="nighttimeLighting"
                                v-model="deviceSettings.nighttimeLightingStart" 
                                timeOnly 
                                hourFormat="12"
                                placeholder="Select time"
                                class="w-full"
                                showIcon
                            />
                        </div>
                    </div>

                    <!-- Level Settings -->
                    <div class="lighting-levels">
                        <div class="form-group mb-4">
                            <label for="ledLevel" class="form-label">LED Lighting Level</label>
                            <div class="slider-container">
                                <Slider 
                                    v-model="deviceSettings.ledLightingLevel" 
                                    :min="0" 
                                    :max="100"
                                    class="w-full mb-2"
                                />
                                <div class="flex justify-between align-items-center">
                                    <InputNumber 
                                        v-model="deviceSettings.ledLightingLevel" 
                                        :min="0" 
                                        :max="100" 
                                        suffix="%" 
                                        class="w-24"
                                        size="small"
                                    />
                                    <small class="text-gray-500">brightness of LED</small>
                                </div>
                            </div>
                        </div>

                        <div class="form-group">
                            <label for="irLevel" class="form-label">IR Lighting Level</label>
                            <div class="slider-container">
                                <Slider 
                                    v-model="deviceSettings.irLightingLevel" 
                                    :min="0" 
                                    :max="100"
                                    class="w-full mb-2"
                                />
                                <div class="flex justify-between align-items-center">
                                    <InputNumber 
                                        v-model="deviceSettings.irLightingLevel" 
                                        :min="0" 
                                        :max="100" 
                                        suffix="%" 
                                        class="w-24"
                                        size="small"
                                    />
                                    <small class="text-gray-500">brightness of IR sensor</small>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Video & System Settings Section -->
            <div class="settings-section mb-6">
                <h3 class="section-title mb-4">
                    <i class="pi pi-video mr-2"></i>
                    Video & System Configuration
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Video Settings -->
                    <div class="video-settings">
                        <div class="sensor-item mb-3">
                            <div class="flex align-items-center">
                                <Checkbox 
                                    id="videoStream" 
                                    v-model="deviceSettings.videoStream" 
                                    :binary="true"
                                    class="mr-3"
                                />
                                <label for="videoStream" class="sensor-label">
                                    <i class="pi pi-play mr-2 text-blue-600"></i>
                                    Video Stream
                                </label>
                            </div>
                        </div>

                        <div class="sensor-item">
                            <div class="flex align-items-center">
                                <Checkbox 
                                    id="videoRecord" 
                                    v-model="deviceSettings.videoRecord" 
                                    :binary="true"
                                    class="mr-3"
                                />
                                <label for="videoRecord" class="sensor-label">
                                    <i class="pi pi-circle mr-2 text-red-600"></i>
                                    Video Record
                                </label>
                            </div>
                        </div>
                    </div>

                    <!-- System Settings -->
                    <div class="system-settings">
                        <div class="form-group">
                            <label for="rebootTime" class="form-label">Daily Reboot Time</label>
                            <Calendar 
                                id="rebootTime"
                                v-model="deviceSettings.dailyRebootTime" 
                                timeOnly 
                                hourFormat="12"
                                placeholder="Select reboot time"
                                class="w-full"
                                showIcon
                            />
                            <small class="text-gray-500 mt-1 block">
                                <i class="pi pi-info-circle mr-1"></i>
                                When should the cage reboot daily?
                            </small>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        </div>

        <template #footer>
            <div class="flex justify-between align-items-center">
                <Button 
                    label="Cancel" 
                    icon="pi pi-times" 
                    @click="cancelSettings" 
                    severity="secondary" 
                    outlined
                    class="mr-3"
                />
                <Button 
                    label="Save Changes" 
                    icon="pi pi-check" 
                    @click="saveSettings" 
                    severity="success"
                />
            </div>
        </template>
    </Dialog>
</template>

<style scoped lang="scss">
.font-mono {
    font-family: 'Courier New', Courier, monospace;
}

/* Settings Dialog Styles */
.settings-dialog {
    .p-dialog-header {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        border-radius: 8px 8px 0 0;
        
        .p-dialog-title {
            font-weight: 600;
            font-size: 1.25rem;
        }
    }
    
    .p-dialog-content {
        padding: 0;
        border-radius: 0 0 8px 8px;
    }
}

.settings-content {
    background: #f8fafc;
    
    .settings-section {
        background: white;
        border-radius: 12px;
        padding: 1.5rem;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
        border: 1px solid #e2e8f0;
        
        .section-title {
            color: #1e293b;
            font-size: 1.125rem;
            font-weight: 600;
            display: flex;
            align-items: center;
            padding-bottom: 0.75rem;
            border-bottom: 2px solid #f1f5f9;
            margin-bottom: 1.5rem;
            
            i {
                color: #3b82f6;
                font-size: 1.25rem;
            }
        }
    }
}

.form-group {
    margin-bottom: 1rem;
    
    .form-label {
        display: block;
        font-weight: 600;
        color: #374151;
        margin-bottom: 0.5rem;
        font-size: 0.875rem;
    }
}

.sensor-item {
    padding: 0.75rem;
    background: #f8fafc;
    border-radius: 8px;
    border: 1px solid #e2e8f0;
    transition: all 0.2s ease;
    
    &:hover {
        background: #f1f5f9;
        border-color: #cbd5e1;
    }
    
    .sensor-label {
        font-weight: 500;
        color: #374151;
        display: flex;
        align-items: center;
        cursor: pointer;
        margin: 0;
        
        i {
            font-size: 1rem;
        }
    }
}

.slider-container {
    .p-slider {
        background: #e2e8f0;
        border-radius: 4px;
        
        .p-slider-range {
            background: linear-gradient(135deg, #3b82f6, #1d4ed8);
        }
        
        .p-slider-handle {
            border: 2px solid #3b82f6;
            background: white;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }
    }
}

.lighting-times,
.lighting-levels,
.video-settings,
.system-settings {
    padding: 1rem;
    background: #f8fafc;
    border-radius: 8px;
    border: 1px solid #e2e8f0;
}

/* Custom input styling */
.p-inputtext,
.p-dropdown,
.p-calendar {
    border-radius: 6px;
    border: 1px solid #d1d5db;
    
    &:focus {
        border-color: #3b82f6;
        box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
    }
}

.p-checkbox {
    .p-checkbox-box {
        border-radius: 4px;
        border: 2px solid #d1d5db;
        
        &.p-highlight {
            background: #3b82f6;
            border-color: #3b82f6;
        }
    }
}

/* Footer styling */
.p-dialog-footer {
    padding: 1.5rem;
    background: #f8fafc;
    border-top: 1px solid #e2e8f0;
    border-radius: 0 0 8px 8px;
}

/* Grid View Styles */
.grid-view {
    .device-card {
        transition: all 0.3s ease;
        
        &:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
        }
        
        .device-image-container {
            position: relative;
            
            &::before {
                content: '';
                position: absolute;
                top: 0;
                left: 0;
                right: 0;
                bottom: 0;
                background: radial-gradient(ellipse at center, transparent 20%, rgba(0, 0, 0, 0.3) 70%);
                pointer-events: none;
            }
            
            /* Simulating camera noise/grain effect */
            &::after {
                content: '';
                position: absolute;
                top: 0;
                left: 0;
                right: 0;
                bottom: 0;
                background-image: 
                    radial-gradient(circle at 25% 25%, rgba(255, 255, 255, 0.1) 1px, transparent 1px),
                    radial-gradient(circle at 75% 75%, rgba(255, 255, 255, 0.1) 1px, transparent 1px);
                background-size: 4px 4px;
                opacity: 0.3;
                pointer-events: none;
            }
        }
    }
}

/* Responsive adjustments */
@media (max-width: 768px) {
    .grid-view {
        .grid {
            grid-template-columns: 1fr;
        }
    }
}

@media (max-width: 1024px) {
    .grid-view {
        .grid {
            grid-template-columns: repeat(2, 1fr);
        }
    }
}
</style>
