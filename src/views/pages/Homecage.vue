<script setup>
import { ref, reactive, onBeforeMount, computed } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();

// Get device ID from route params
const deviceId = computed(() => route.params.id || 'LETH02C001');

// Sample devices data to find the current device
const allDevices = [
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
];

// Find current device
const currentDevice = computed(() => {
    return allDevices.find(device => device.hostname === deviceId.value) || allDevices[0];
});

// Reactive data for the homecage
const homecageData = reactive({
    hostname: '',
    macAddress: '',
    location: '',
    ipAddress: '',
    version: '',
    remoteStorage: 'Not Available'
});

// Environment data
const environmentData = reactive({
    humidity: { value: 'True', status: 'success' },
    temperature: { value: 'N/A', status: 'info' },
    waterLevel: 100,
    foodLevel: 99.98,
    ammoniaLevel: 0.67
});

// Tags
const tags = ref([]);

// Video status
const videoStatus = reactive({
    isRecording: false,
    hasVideo: true
});

// Chart data for sensors
const chartData = ref({
    temperatureHumidity: {
        labels: ['Jul 27', 'Jul 28', 'Jul 29', 'Jul 30'],
        datasets: [
            {
                label: 'Temperature Sensor',
                data: [3.8, 3.6, 3.4, 3.2],
                borderColor: '#FF6B35',
                backgroundColor: '#FF6B35',
                tension: 0.4
            },
            {
                label: 'Humidity Sensor',
                data: [3.2, 3.0, 2.8, 2.6],
                borderColor: '#1E40AF',
                backgroundColor: '#1E40AF',
                tension: 0.4
            }
        ]
    },
    foodSensor: {
        labels: ['Jul 27', 'Jul 28', 'Jul 29', 'Jul 30'],
        datasets: [
            {
                label: 'Food Sensor',
                data: [1.0, 0.8, 0.6, 0.4],
                borderColor: '#4338CA',
                backgroundColor: '#4338CA',
                tension: 0.4
            }
        ]
    },
    waterAmmoniaReading: {
        labels: ['Jul 27', 'Jul 28', 'Jul 29', 'Jul 30'],
        datasets: [
            {
                label: 'Water Sensor',
                data: [2.1, 1.8, 1.5, 1.2],
                borderColor: '#1E40AF',
                backgroundColor: '#1E40AF',
                tension: 0.4
            },
            {
                label: 'Ammonia Sensor',
                data: [1.0, 0.8, 0.6, 0.4],
                borderColor: '#059669',
                backgroundColor: '#059669',
                tension: 0.4
            }
        ]
    }
});

// Chart options
const chartOptions = ref({
    responsive: true,
    maintainAspectRatio: false,
    plugins: {
        legend: {
            position: 'bottom'
        }
    },
    scales: {
        y: {
            beginAtZero: true,
            max: 4.2
        }
    }
});

// Date range for state data
const startingDate = ref(new Date('2025-07-22'));
const endingDate = ref(new Date('2025-07-29'));

// Functions
function addTag() {
    // Add tag functionality
    console.log('Add tag clicked');
}

function editField(field) {
    console.log('Edit field:', field);
}

function deleteCage() {
    console.log('Delete cage clicked');
}

function playVideo() {
    console.log('Play video clicked');
}

function toggleRecording() {
    videoStatus.isRecording = !videoStatus.isRecording;
    console.log('Recording status:', videoStatus.isRecording);
}

function submitStateData() {
    console.log('Submit state data:', {
        startingDate: startingDate.value,
        endingDate: endingDate.value
    });
}

function resetStateData() {
    startingDate.value = new Date('2025-07-22');
    endingDate.value = new Date('2025-07-29');
}

function goToNext() {
    const currentIndex = allDevices.findIndex(device => device.hostname === deviceId.value);
    const nextIndex = (currentIndex + 1) % allDevices.length;
    const nextDevice = allDevices[nextIndex];
    router.push({ name: 'homecage', params: { id: nextDevice.hostname } });
}

onBeforeMount(() => {
    // Initialize data with current device
    updateHomecageData();
    console.log('Homecage page loaded for:', deviceId.value);
});

function updateHomecageData() {
    const device = currentDevice.value;
    homecageData.hostname = device.hostname;
    homecageData.macAddress = device.macAddress;
    homecageData.location = device.location;
    homecageData.ipAddress = device.ipAddress;
    homecageData.version = device.softwareVersion;
}

// Watch for route changes
import { watch } from 'vue';
watch(() => route.params.id, () => {
    updateHomecageData();
});
</script>

<template>
    <div class="homecage-container">
        <!-- Header -->
        <div class="flex items-center justify-between mb-6">
            <div class="flex items-center gap-4">
                <h1 class="text-2xl font-bold">HomeCage #{{ homecageData.hostname }}</h1>
                <Button label="Next" icon="pi pi-chevron-right" iconPos="right" size="small" outlined @click="goToNext" />
            </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <!-- Left Column -->
            <div class="space-y-6">
                <!-- Cage Information Card -->
                <div class="card">
                    <div class="flex items-center gap-2 mb-4">
                        <i class="pi pi-info-circle text-blue-500"></i>
                        <h3 class="font-semibold text-lg">Cage Information</h3>
                    </div>
                    
                    <div class="grid grid-cols-2 gap-4">
                        <div class="info-item">
                            <label class="text-sm text-gray-600">Hostname</label>
                            <div class="flex items-center gap-2">
                                <span class="font-mono">{{ homecageData.hostname }}</span>
                                <Button icon="pi pi-pencil" size="small" text @click="editField('hostname')" />
                            </div>
                        </div>
                        
                        <div class="info-item">
                            <label class="text-sm text-gray-600">MAC Address</label>
                            <div class="flex items-center gap-2">
                                <span class="font-mono">{{ homecageData.macAddress }}</span>
                                <Button icon="pi pi-pencil" size="small" text @click="editField('macAddress')" />
                            </div>
                        </div>
                        
                        <div class="info-item">
                            <label class="text-sm text-gray-600">Location</label>
                            <div class="flex items-center gap-2">
                                <span>{{ homecageData.location }}</span>
                                <Button icon="pi pi-pencil" size="small" text @click="editField('location')" />
                            </div>
                        </div>
                        
                        <div class="info-item">
                            <label class="text-sm text-gray-600">IP Address</label>
                            <div class="flex items-center gap-2">
                                <span class="font-mono">{{ homecageData.ipAddress }}</span>
                                <Button icon="pi pi-pencil" size="small" text @click="editField('ipAddress')" />
                            </div>
                        </div>
                        
                        <div class="info-item">
                            <label class="text-sm text-gray-600">Version</label>
                            <div class="flex items-center gap-2">
                                <span class="font-mono">{{ homecageData.version }}</span>
                                <Button icon="pi pi-pencil" size="small" text @click="editField('version')" />
                            </div>
                        </div>
                        
                        <div class="info-item">
                            <label class="text-sm text-gray-600">Remote Storage</label>
                            <div class="flex items-center gap-2">
                                <span class="text-gray-500 italic">{{ homecageData.remoteStorage }}</span>
                                <Button icon="pi pi-pencil" size="small" text @click="editField('remoteStorage')" />
                            </div>
                        </div>
                    </div>

                    <!-- Tags Section -->
                    <div class="mt-6">
                        <div class="flex items-center justify-between mb-2">
                            <label class="text-sm text-gray-600">Tags</label>
                            <Button label="+ Add Tag" size="small" text @click="addTag" />
                        </div>
                        <p class="text-sm text-gray-500 italic" v-if="tags.length === 0">No tags assigned</p>
                        <div class="flex gap-2 flex-wrap" v-else>
                            <Tag v-for="tag in tags" :key="tag" :value="tag" />
                        </div>
                    </div>

                    <!-- Action Buttons -->
                    <div class="flex gap-3 mt-6">
                        <Button icon="pi pi-trash" label="Delete Cage" severity="danger" outlined @click="deleteCage" />
                        <Button icon="pi pi-play" label="Video Player" @click="playVideo" />
                        <Button 
                            :icon="videoStatus.isRecording ? 'pi pi-stop' : 'pi pi-circle'" 
                            :label="videoStatus.isRecording ? 'Stop Recording' : 'Video Record Status'"
                            :severity="videoStatus.isRecording ? 'danger' : 'secondary'"
                            @click="toggleRecording" 
                        />
                    </div>
                </div>

                
            </div>

            <!-- Right Column -->
            <div class="space-y-6">
                <!-- Cage Stream Card -->
                <div class="card">
                    <div class="flex items-center gap-2 mb-4">
                        <i class="pi pi-video text-blue-500"></i>
                        <h3 class="font-semibold text-lg">Cage Stream</h3>
                    </div>
                    
                    <div class="bg-gray-100 rounded-lg h-48 flex items-center justify-center">
                        <div class="text-center">
                            <i class="pi pi-play-circle text-4xl text-gray-400 mb-2"></i>
                            <p class="text-gray-600">📹 Video for HomeCage #{{ homecageData.hostname }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Environment Card -->
                <div class="card mt-6">
                    <div class="flex items-center gap-2 mb-4">
                        <i class="pi pi-cloud text-blue-500"></i>
                        <h3 class="font-semibold text-lg">Environment</h3>
                    </div>
                    
                    <div class="grid grid-cols-2 gap-4 mb-6">
                        <div class="text-center">
                            <label class="text-sm text-gray-600">Humidity</label>
                            <div class="mt-1">
                                <Tag :value="environmentData.humidity.value" :severity="environmentData.humidity.status" />
                            </div>
                        </div>
                        <div class="text-center">
                            <label class="text-sm text-gray-600">Temperature</label>
                            <div class="mt-1">
                                <Tag :value="environmentData.temperature.value" :severity="environmentData.temperature.status" />
                            </div>
                        </div>
                    </div>

                    <!-- Circular Progress Charts -->
                    <div class="grid grid-cols-3 gap-4">
                        <div class="text-center">
                            <label class="text-sm text-gray-600 block mb-2">Water Level</label>
                            <div class="relative inline-block">
                                <svg width="80" height="80" class="transform -rotate-90">
                                    <circle cx="40" cy="40" r="35" stroke="#e5e7eb" stroke-width="6" fill="none"/>
                                    <circle 
                                        cx="40" cy="40" r="35" 
                                        stroke="#10b981" 
                                        stroke-width="6" 
                                        fill="none"
                                        :stroke-dasharray="219.9"
                                        :stroke-dashoffset="219.9 * (1 - environmentData.waterLevel / 100)"
                                        stroke-linecap="round"
                                    />
                                </svg>
                                <div class="absolute inset-0 flex items-center justify-center">
                                    <span class="text-sm font-semibold">{{ environmentData.waterLevel }}%</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="text-center">
                            <label class="text-sm text-gray-600 block mb-2">Food Level</label>
                            <div class="relative inline-block">
                                <svg width="80" height="80" class="transform -rotate-90">
                                    <circle cx="40" cy="40" r="35" stroke="#e5e7eb" stroke-width="6" fill="none"/>
                                    <circle 
                                        cx="40" cy="40" r="35" 
                                        stroke="#10b981" 
                                        stroke-width="6" 
                                        fill="none"
                                        :stroke-dasharray="219.9"
                                        :stroke-dashoffset="219.9 * (1 - environmentData.foodLevel / 100)"
                                        stroke-linecap="round"
                                    />
                                </svg>
                                <div class="absolute inset-0 flex items-center justify-center">
                                    <span class="text-sm font-semibold">{{ environmentData.foodLevel }}%</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="text-center">
                            <label class="text-sm text-gray-600 block mb-2">Ammonia Level</label>
                            <div class="relative inline-block">
                                <svg width="80" height="80" class="transform -rotate-90">
                                    <circle cx="40" cy="40" r="35" stroke="#e5e7eb" stroke-width="6" fill="none"/>
                                    <circle 
                                        cx="40" cy="40" r="35" 
                                        stroke="#9ca3af" 
                                        stroke-width="6" 
                                        fill="none"
                                        :stroke-dasharray="219.9"
                                        :stroke-dashoffset="219.9 * (1 - environmentData.ammoniaLevel / 100)"
                                        stroke-linecap="round"
                                    />
                                </svg>
                                <div class="absolute inset-0 flex items-center justify-center">
                                    <span class="text-sm font-semibold">{{ environmentData.ammoniaLevel }}%</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
        
        <!-- State Data Section -->
        <div class="card mt-6">
            <div class="flex items-center gap-2 mb-4">
                <i class="pi pi-chart-line text-blue-500"></i>
                <h3 class="font-semibold text-lg">Homecage State Data</h3>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
                <div>
                    <label class="block text-sm text-gray-600 mb-2">Starting Date*</label>
                    <Calendar v-model="startingDate" showIcon dateFormat="mm/dd/yy" />
                    <small class="text-gray-500">Select the date you wish to set as starting point for plotting.</small>
                </div>
                <div>
                    <label class="block text-sm text-gray-600 mb-2">Ending Date*</label>
                    <Calendar v-model="endingDate" showIcon dateFormat="mm/dd/yy" />
                    <small class="text-gray-500">Select the date you wish to set as ending point for plotting.</small>
                </div>
            </div>

            <div class="flex gap-3 mb-6">
                <Button label="Submit" icon="pi pi-check" @click="submitStateData" />
                <Button label="Reset" icon="pi pi-refresh" severity="secondary" outlined @click="resetStateData" />
            </div>

            <!-- Charts Grid -->
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <!-- Temperature and Humidity Chart -->
                <div class="chart-container">
                    <h4 class="font-semibold mb-3">Temperature and Humidity Sensors</h4>
                    <Chart type="line" :data="chartData.temperatureHumidity" :options="chartOptions" class="h-64" />
                </div>

                <!-- Food Sensor Chart -->
                <div class="chart-container">
                    <h4 class="font-semibold mb-3">Food Sensor</h4>
                    <Chart type="line" :data="chartData.foodSensor" :options="chartOptions" class="h-64" />
                </div>

                <!-- Water and Ammonia Chart -->
                <div class="chart-container lg:col-span-2">
                    <h4 class="font-semibold mb-3">Water and Ammonia Reading</h4>
                    <Chart type="line" :data="chartData.waterAmmoniaReading" :options="chartOptions" class="h-64" />
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped lang="scss">
.homecage-container {
    padding: 1rem;
}

.card {
    background: white;
    border-radius: 8px;
    padding: 1.5rem;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    border: 1px solid #e5e7eb;
}

.info-item {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
}

.info-item label {
    font-weight: 500;
}

.font-mono {
    font-family: 'Courier New', Courier, monospace;
}

.chart-container {
    background: #f9fafb;
    border-radius: 6px;
    padding: 1rem;
}

// Circular progress animation
circle {
    transition: stroke-dashoffset 0.5s ease-in-out;
}
</style>
