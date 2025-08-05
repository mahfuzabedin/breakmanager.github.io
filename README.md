<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Team Break Manager (Hybrid)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .animate-fade-in {
            animation: fadeIn 0.5s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .time-slot {
            cursor: pointer;
            transition: background-color 0.2s;
        }
        .time-slot.available:hover {
            background-color: #cceeff; /* A light blue for hover */
        }
        .time-slot.booked { background-color: #fde68a; /* yellow-200 */ }
        .time-slot.pending { background-color: #d8b4fe; /* purple-300 */ color: #581c87; }
        .time-slot.on-break { background-color: #fca5a5; /* red-400 */ color: white; }
        .time-slot.ad-hoc { background-color: #93c5fd; /* blue-300 */ color: #1e3a8a; }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div class="container mx-auto p-4 sm:p-6 md:p-8 max-w-7xl">
        <header class="text-center mb-8">
            <h1 class="text-3xl sm:text-4xl font-bold text-blue-600">Team Break Manager</h1>
            <p class="text-gray-500 mt-2">Use the Live Roster for immediate breaks or the schedule to book in advance.</p>
        </header>

        <!-- Top Row: Controls & Live Status -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
            <!-- Manager Controls -->
            <div class="bg-white p-6 rounded-xl shadow-md animate-fade-in">
                <h2 class="text-xl font-bold mb-4 text-gray-700 border-b pb-2">Manager Controls</h2>
                <div class="flex flex-col sm:flex-row items-center space-y-4 sm:space-y-0 sm:space-x-4 mb-6">
                    <label for="break-limit" class="font-semibold">Set Break Limit:</label>
                    <input type="number" id="break-limit" value="4" min="1" class="w-24 p-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                    <button id="set-limit-btn" class="w-full sm:w-auto bg-blue-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-blue-700">Set</button>
                </div>
                <div id="manager-requests-panel">
                     <h3 class="text-lg font-bold text-gray-700 border-b pb-2 mb-3">Pending Requests</h3>
                     <div id="pending-requests-list" class="space-y-2 max-h-40 overflow-y-auto">
                        <p class="text-gray-500 italic">No pending requests.</p>
                     </div>
                </div>
            </div>
            <!-- Live Status & Roster -->
            <div class="bg-white p-6 rounded-xl shadow-md animate-fade-in">
                 <h2 class="text-xl font-bold mb-4 text-gray-700 border-b pb-2">Live Roster & Status</h2>
                 <div class="grid grid-cols-2 gap-4 mb-4 text-center">
                    <div class="bg-green-100 p-3 rounded-lg">
                        <p class="font-semibold text-green-800">Available</p>
                        <p id="available-count" class="text-3xl font-bold text-green-600">0</p>
                    </div>
                    <div class="bg-yellow-100 p-3 rounded-lg">
                        <p class="font-semibold text-yellow-800">On Break</p>
                        <p id="on-break-count" class="text-3xl font-bold text-yellow-600">0 / 4</p>
                    </div>
                </div>
                <div id="live-roster-list" class="space-y-2 max-h-48 overflow-y-auto">
                    <!-- Live roster for ad-hoc breaks inserted here -->
                </div>
            </div>
        </div>

        <!-- Break Booking Schedule -->
        <div class="bg-white rounded-xl shadow-md animate-fade-in overflow-x-auto">
             <h2 class="text-xl font-bold text-gray-700 border-b pb-2 p-6">Break Booking Schedule</h2>
            <table class="min-w-full text-sm text-left">
                <thead class="bg-gray-50">
                    <tr>
                        <th class="p-3 font-semibold text-gray-600 w-48 sticky left-0 bg-gray-50 z-10">Team Member</th>
                        <!-- Time slots will be dynamically inserted here -->
                    </tr>
                </thead>
                <tbody id="schedule-body">
                    <!-- Schedule rows will be dynamically inserted here -->
                </tbody>
            </table>
        </div>
    </div>

    <!-- Authorization Modal -->
    <div id="auth-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 hidden animate-fade-in z-50">
        <div class="bg-white rounded-xl shadow-2xl p-8 max-w-md w-full text-center">
            <div class="mx-auto flex items-center justify-center h-12 w-12 rounded-full bg-red-100">
                <svg class="h-6 w-6 text-red-600" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" /></svg>
            </div>
            <h3 class="text-xl font-bold mt-4 text-gray-800">Break Limit Reached</h3>
            <p class="text-gray-600 mt-2 mb-6">The maximum number of people are on break. You can request authorization from your manager.</p>
            <div class="flex justify-center space-x-4">
                 <button id="wait-slot-btn" class="w-full bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded-lg hover:bg-gray-300">Wait for Slot</button>
                 <button id="request-auth-btn" class="w-full bg-blue-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-blue-700">Send Request</button>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // --- STATE MANAGEMENT ---
            let teamData = [
                { id: 1, name: 'Mehreen', breaks: [], status: 'available' },
                { id: 2, name: 'Nazat', breaks: [], status: 'available' },
                { id: 3, name: 'Sabit', breaks: [{time: '10:30', status: 'booked'}, {time: '10:45', status: 'booked'}], status: 'available' },
                { id: 4, name: 'Zafrin', breaks: [], status: 'available' },
                { id: 5, name: 'Iftekhar', breaks: [], status: 'available' },
                { id: 6, name: 'Jumon', breaks: [], status: 'available' },
                { id: 7, name: 'Mahfuz', breaks: [], status: 'available' },
                { id: 8, name: 'Talal', breaks: [], status: 'available' },
            ];
            let breakLimit = 4;
            let pendingRequests = [];
            let currentRequest = null;
            
            const timeSlots = Array.from({length: 36}, (_, i) => {
                const h = 9 + Math.floor(i / 4);
                const m = (i % 4) * 15;
                return `${String(h).padStart(2, '0')}:${String(m).padStart(2, '0')}`;
            });

            // --- DOM ELEMENTS ---
            const scheduleHeader = document.querySelector('thead tr');
            const scheduleBody = document.getElementById('schedule-body');
            const liveRosterList = document.getElementById('live-roster-list');
            const breakLimitInput = document.getElementById('break-limit');
            const setLimitBtn = document.getElementById('set-limit-btn');
            const availableCountEl = document.getElementById('available-count');
            const onBreakCountEl = document.getElementById('on-break-count');
            const authModal = document.getElementById('auth-modal');
            const requestAuthBtn = document.getElementById('request-auth-btn');
            const waitSlotBtn = document.getElementById('wait-slot-btn');
            const pendingRequestsList = document.getElementById('pending-requests-list');

            // --- RENDER FUNCTIONS ---
            const renderAll = () => {
                const onBreakCount = updateLiveStatus();
                renderSchedule(onBreakCount);
                renderLiveRoster();
                renderPendingRequests();
            };
            
            const renderLiveRoster = () => {
                liveRosterList.innerHTML = '';
                teamData.forEach(member => {
                    const isAvailable = member.status === 'available';
                    const rosterDiv = document.createElement('div');
                    rosterDiv.className = `p-2 rounded-lg flex items-center justify-between transition duration-300 ${isAvailable ? 'bg-gray-50' : 'bg-blue-100'}`;
                    rosterDiv.innerHTML = `
                        <div class="flex items-center">
                            <div class="w-2 h-2 rounded-full mr-3 ${isAvailable ? 'bg-green-500' : 'bg-blue-500'}"></div>
                            <span class="font-semibold">${member.name}</span>
                        </div>
                        <button data-id="${member.id}" class="toggle-adhoc-btn w-28 text-sm text-white font-bold py-1 px-2 rounded-md ${isAvailable ? 'bg-green-500 hover:bg-green-600' : 'bg-blue-500 hover:bg-blue-600'}">
                            ${isAvailable ? 'Start Break' : 'End Break'}
                        </button>
                    `;
                    liveRosterList.appendChild(rosterDiv);
                });
            };

            const renderSchedule = (onBreakCount) => {
                if (!scheduleHeader.querySelector('.time-header')) {
                    timeSlots.forEach(slot => {
                        const th = document.createElement('th');
                        th.className = 'p-2 font-semibold text-gray-600 border-l time-header';
                        th.textContent = slot;
                        scheduleHeader.appendChild(th);
                    });
                }

                scheduleBody.innerHTML = '';
                teamData.forEach(member => {
                    const tr = document.createElement('tr');
                    tr.className = 'border-t';
                    let rowHtml = `<td class="p-3 font-bold text-gray-700 sticky left-0 bg-white border-r">${member.name}</td>`;
                    
                    timeSlots.forEach(slot => {
                        const breakInfo = member.breaks.find(b => b.time === slot);
                        let slotClass = 'available';
                        let slotText = '';
                        if (breakInfo) {
                            slotClass = breakInfo.status;
                            if(breakInfo.status !== 'available') {
                                slotText = breakInfo.status.charAt(0).toUpperCase() + breakInfo.status.slice(1);
                            }
                        }
                        rowHtml += `<td class="p-0 border-l text-center time-slot ${slotClass}" data-member-id="${member.id}" data-time="${slot}">${slotText}</td>`;
                    });
                    tr.innerHTML = rowHtml;
                    scheduleBody.appendChild(tr);
                });
            };

            const renderPendingRequests = () => {
                pendingRequestsList.innerHTML = '';
                if (pendingRequests.length === 0) {
                    pendingRequestsList.innerHTML = '<p class="text-gray-500 italic">No pending requests.</p>';
                    return;
                }
                pendingRequests.forEach(req => {
                    const member = teamData.find(m => m.id === req.memberId);
                    const reqDiv = document.createElement('div');
                    reqDiv.className = 'flex justify-between items-center p-2 bg-gray-100 rounded-lg';
                    reqDiv.innerHTML = `
                        <span><strong>${member.name}</strong> @ <strong>${req.time}</strong></span>
                        <div class="space-x-2">
                            <button data-request-id="${req.id}" class="deny-btn bg-red-500 text-white px-3 py-1 rounded text-sm">Deny</button>
                            <button data-request-id="${req.id}" class="approve-btn bg-green-500 text-white px-3 py-1 rounded text-sm">Approve</button>
                        </div>
                    `;
                    pendingRequestsList.appendChild(reqDiv);
                });
            };

            // --- LOGIC FUNCTIONS ---
            const updateLiveStatus = () => {
                const now = new Date();
                const currentTimeSlot = `${String(now.getHours()).padStart(2, '0')}:${String(Math.floor(now.getMinutes() / 15) * 15).padStart(2, '0')}`;
                
                let onBreakCount = 0;
                teamData.forEach(member => {
                    const isOnScheduledBreak = member.breaks.some(b => b.time === currentTimeSlot && (b.status === 'booked' || b.status === 'on-break' || b.status === 'ad-hoc'));
                    if (member.status === 'on-ad-hoc-break' || isOnScheduledBreak) {
                        onBreakCount++;
                    }
                });
                
                const availableCount = teamData.length - onBreakCount;
                availableCountEl.textContent = availableCount;
                onBreakCountEl.textContent = `${onBreakCount} / ${breakLimit}`;

                if (onBreakCount >= breakLimit) {
                    onBreakCountEl.parentElement.classList.add('bg-red-100', 'text-red-600');
                    onBreakCountEl.parentElement.classList.remove('bg-yellow-100', 'text-yellow-800');
                } else {
                    onBreakCountEl.parentElement.classList.add('bg-yellow-100', 'text-yellow-800');
                    onBreakCountEl.parentElement.classList.remove('bg-red-100', 'text-red-600');
                }
                return onBreakCount;
            };
            
            const handleSlotClick = (memberId, time) => {
                const member = teamData.find(m => m.id === memberId);
                const existingBreak = member.breaks.find(b => b.time === time);
                
                if (existingBreak) {
                    if (confirm('Do you want to cancel this break/request?')) {
                        member.breaks = member.breaks.filter(b => b.time !== time);
                        if (existingBreak.status === 'pending') {
                            pendingRequests = pendingRequests.filter(r => !(r.memberId === memberId && r.time === time));
                        }
                    }
                } else {
                    let concurrentBreaks = 0;
                    teamData.forEach(m => {
                        if (m.breaks.some(b => b.time === time && b.status !== 'pending')) concurrentBreaks++;
                    });
                    if (concurrentBreaks >= breakLimit) {
                        currentRequest = { memberId, time, type: 'scheduled' };
                        authModal.classList.remove('hidden');
                        return;
                    }
                    member.breaks.push({ time: time, status: 'booked' });
                }
                renderAll();
            };

            const handleAdHocToggle = (memberId) => {
                const member = teamData.find(m => m.id === memberId);
                if (member.status === 'available') { // Start break
                    const onBreakCount = updateLiveStatus();
                    if (onBreakCount >= breakLimit) {
                        const now = new Date();
                        const time = `${String(now.getHours()).padStart(2, '0')}:${String(Math.floor(now.getMinutes() / 15) * 15).padStart(2, '0')}`;
                        currentRequest = { memberId, time, type: 'ad-hoc' };
                        authModal.classList.remove('hidden');
                        return;
                    }
                    member.status = 'on-ad-hoc-break';
                } else { // End break
                    member.status = 'available';
                }
                renderAll();
            };

            const handleRequestAuth = () => {
                if (!currentRequest) return;
                const { memberId, time } = currentRequest;
                const requestId = Date.now();
                pendingRequests.push({ id: requestId, memberId, time, type: currentRequest.type });
                
                if (currentRequest.type === 'scheduled') {
                    const member = teamData.find(m => m.id === memberId);
                    member.breaks.push({ time: time, status: 'pending' });
                }
                
                authModal.classList.add('hidden');
                currentRequest = null;
                renderAll();
            };

            const handleManagerAction = (e) => {
                if (!e.target.matches('.approve-btn, .deny-btn')) return;
                const requestId = parseInt(e.target.dataset.requestId);
                const request = pendingRequests.find(r => r.id === requestId);
                if (!request) return;

                const member = teamData.find(m => m.id === request.memberId);
                
                if (e.target.matches('.approve-btn')) {
                    if (request.type === 'scheduled') {
                        const breakToUpdate = member.breaks.find(b => b.time === request.time && b.status === 'pending');
                        if (breakToUpdate) breakToUpdate.status = 'booked';
                    } else { // ad-hoc
                        member.status = 'on-ad-hoc-break';
                    }
                } else { // Deny
                    const breakToUpdate = member.breaks.find(b => b.time === request.time && b.status === 'pending');
                    if (breakToUpdate) member.breaks = member.breaks.filter(b => b !== breakToUpdate);
                }

                pendingRequests = pendingRequests.filter(r => r.id !== requestId);
                renderAll();
            };

            // --- EVENT LISTENERS ---
            setLimitBtn.addEventListener('click', () => {
                breakLimit = parseInt(breakLimitInput.value, 10) || breakLimit;
                renderAll();
            });
            scheduleBody.addEventListener('click', (e) => {
                if (e.target.classList.contains('time-slot')) handleSlotClick(parseInt(e.target.dataset.memberId), e.target.dataset.time);
            });
            liveRosterList.addEventListener('click', (e) => {
                if (e.target.classList.contains('toggle-adhoc-btn')) handleAdHocToggle(parseInt(e.target.dataset.id));
            });
            requestAuthBtn.addEventListener('click', handleRequestAuth);
            waitSlotBtn.addEventListener('click', () => authModal.classList.add('hidden'));
            pendingRequestsList.addEventListener('click', handleManagerAction);

            // --- INITIALIZATION ---
            renderAll();
            setInterval(renderAll, 30000); // Refresh every 30 seconds
        });
    </script>
</body>
</html>
