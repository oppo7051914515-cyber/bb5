<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Check-In Online - ระบบเช็คชื่อนักเรียนออนไลน์</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: { prompt: ['Prompt', 'sans-serif'] }
                }
            }
        }
    </script>
    <style>
        body { font-family: 'Prompt', sans-serif; }
        .glass-card {
            background: rgba(255, 255, 255, 0.98);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(226, 232, 240, 0.8);
        }
        .pulse-slow {
            animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: .5; }
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col selection:bg-blue-500 selection:text-white">

    <!-- Header -->
    <header id="main-header" class="bg-slate-900 text-white shadow-lg sticky top-0 z-40 border-b border-slate-800">
        <div class="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-3 cursor-pointer" onclick="switchTab('dashboard')">
                <div class="bg-gradient-to-tr from-blue-600 to-indigo-600 p-2.5 rounded-xl text-white shadow-md shadow-blue-500/20 flex items-center justify-center">
                    <i class="fa-solid fa-qrcode text-xl"></i>
                </div>
                <div>
                    <h1 class="font-bold text-lg leading-tight flex items-center">
                        Smart Check-In
                        <span class="ml-2 text-[10px] bg-emerald-500/20 text-emerald-400 border border-emerald-500/30 font-semibold px-2 py-0.5 rounded-full">v4.1 Fixed</span>
                    </h1>
                    <p class="text-xs text-slate-400">ระบบเช็คชื่อข้ามอุปกรณ์ Real-time</p>
                </div>
            </div>
            
            <div id="teacher-nav" class="hidden md:flex space-x-1 text-sm font-medium">
                <button onclick="switchTab('dashboard')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-blue-400" id="nav-dashboard">
                    <i class="fa-solid fa-chalkboard-user mr-1.5"></i>เปิดคาบเรียน
                </button>
                <button onclick="switchTab('classes')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-classes">
                    <i class="fa-solid fa-school mr-1.5"></i>จัดการห้องเรียน
                </button>
                <button onclick="switchTab('students')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-students">
                    <i class="fa-solid fa-users mr-1.5"></i>จัดการนักเรียน
                </button>
                <button onclick="switchTab('reports')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-reports">
                    <i class="fa-solid fa-chart-line mr-1.5"></i>รายงานสถิติ
                </button>
            </div>

            <div class="flex items-center space-x-2">
                <span id="sync-status" class="inline-flex items-center text-xs px-3 py-1.5 rounded-full bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 font-medium">
                    <span class="w-2 h-2 rounded-full bg-emerald-400 mr-1.5 pulse-slow"></span>
                    <span>Direct Active</span>
                </span>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto p-4 md:p-6">

        <!-- STUDENT VIEW -->
        <div id="student-view" class="hidden max-w-md mx-auto py-4">
            <div class="text-center mb-6">
                <div class="inline-flex p-3 bg-blue-600 text-white rounded-2xl shadow-lg shadow-blue-500/30 mb-2">
                    <i class="fa-solid fa-qrcode text-3xl"></i>
                </div>
                <h1 class="text-xl font-bold text-slate-800">ระบบเช็คชื่อนักเรียนออนไลน์</h1>
                <p class="text-xs text-slate-500">เลือกชื่อของคุณเพื่อขอรหัส OTP เข้าเรียน</p>
            </div>

            <div class="glass-card rounded-2xl shadow-xl p-6 text-center border-t-4 border-blue-600 relative overflow-hidden">
                <div id="student-state-loading" class="py-8 space-y-3">
                    <i class="fa-solid fa-circle-notch animate-spin text-4xl text-blue-600"></i>
                    <h3 class="text-base font-bold text-slate-700">กำลังเชื่อมต่อรายชื่อ...</h3>
                </div>

                <div id="student-state-closed" class="hidden py-4">
                    <div class="w-16 h-16 bg-rose-50 text-rose-600 rounded-2xl flex items-center justify-center mx-auto mb-4 text-2xl shadow-inner border border-rose-100">
                        <i class="fa-solid fa-triangle-exclamation"></i>
                    </div>
                    <h2 class="text-xl font-bold text-slate-800">ไม่พบคาบเรียน หรือ คาบเรียนถูกปิดแล้ว</h2>
                    <p class="text-xs text-slate-500 mt-2 mb-4">โปรดแจ้งคุณครูผู้สอนให้ตรวจสอบการเปิดคาบเรียน</p>
                    <button onclick="fetchStudentSessionData()" class="inline-flex items-center text-xs bg-blue-50 hover:bg-blue-100 text-blue-600 font-semibold px-4 py-2 rounded-xl transition border border-blue-200">
                        <i class="fa-solid fa-rotate mr-1.5"></i> ลองโหลดข้อมูลใหม่
                    </button>
                </div>

                <div id="student-state-active" class="hidden">
                    <div class="w-16 h-16 bg-blue-50 text-blue-600 rounded-2xl flex items-center justify-center mx-auto mb-4 text-2xl shadow-inner border border-blue-100">
                        <i class="fa-solid fa-user-check"></i>
                    </div>
                    <h2 class="text-xl font-bold text-slate-800" id="student-class-title">ห้องเรียน</h2>
                    <p class="text-xs text-slate-500 mt-1 mb-6 font-medium" id="student-session-info">วิชา: -</p>

                    <div id="student-step-select" class="space-y-4">
                        <div class="text-left">
                            <label class="block text-xs font-bold text-slate-700 uppercase mb-2">
                                เลือกชื่อ-นามสกุลของคุณ
                            </label>
                            <select id="student-dropdown" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-3.5 text-slate-800 font-medium focus:ring-2 focus:ring-blue-500 focus:outline-none shadow-sm text-sm">
                                <option value="">-- ดึงข้อมูลรายชื่อ... --</option>
                            </select>
                        </div>

                        <button onclick="requestStudentOTP()" class="w-full bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-700 hover:to-indigo-700 active:scale-95 text-white font-semibold py-3.5 px-4 rounded-xl shadow-lg shadow-blue-500/30 transition duration-200 flex items-center justify-center space-x-2 text-sm">
                            <i class="fa-solid fa-key"></i>
                            <span>ขอรับรหัส OTP 6 หลัก</span>
                        </button>
                    </div>

                    <div id="student-step-otp" class="hidden mt-4 bg-slate-900 text-white rounded-2xl p-6 shadow-xl relative overflow-hidden text-center">
                        <span class="inline-flex items-center bg-amber-500/20 text-amber-300 text-xs px-3 py-1 rounded-full font-medium mb-3 border border-amber-500/30">
                            <i class="fa-solid fa-circle-notch animate-spin mr-1.5"></i>รอคุณครูอนุมัติรหัส
                        </span>
                        <p class="text-xs text-slate-400">แจ้งรหัส 6 หลักนี้แก่คุณครูเพื่อยืนยัน:</p>
                        <div class="text-4xl font-black tracking-widest text-amber-400 my-3 font-mono drop-shadow-md" id="display-otp-code">------</div>
                    </div>

                    <div id="student-step-success" class="hidden mt-4 bg-emerald-50 text-emerald-900 rounded-2xl p-6 border border-emerald-200 shadow-md text-center space-y-2">
                        <div class="w-16 h-16 bg-emerald-500 text-white rounded-full flex items-center justify-center mx-auto text-2xl shadow-lg shadow-emerald-500/30">
                            <i class="fa-solid fa-check text-3xl"></i>
                        </div>
                        <h3 class="font-bold text-2xl text-emerald-800">เช็คชื่อสำเร็จ! 🟢</h3>
                        <p class="text-xs text-emerald-700 font-medium">คุณครูยืนยันการเข้าเรียนเรียบร้อยแล้ว</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: DASHBOARD -->
        <div id="tab-dashboard" class="tab-content space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 items-end">
                    <div>
                        <label class="block text-xs font-bold text-slate-600 mb-1.5 uppercase tracking-wide">1. เลือกห้องเรียน</label>
                        <select id="teacher-class-select" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-3 text-slate-800 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                            <option value="">-- เลือกห้องเรียน --</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-600 mb-1.5 uppercase tracking-wide">2. ชื่อวิชา / คาบเรียน</label>
                        <input type="text" id="teacher-subject-input" placeholder="เช่น วิทยาการคำนวณ คาบ 1" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-3 text-slate-800 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                    </div>
                    <div>
                        <button id="btn-toggle-session" onclick="toggleClassSession()" class="w-full bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow-md transition duration-200 flex items-center justify-center space-x-2 text-sm">
                            <i class="fa-solid fa-play"></i>
                            <span>🟢 เริ่มเปิดคาบเรียน (Real-time)</span>
                        </button>
                    </div>
                </div>
            </div>

            <div id="active-session-container" class="hidden grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="lg:col-span-1 space-y-6">
                    <div class="glass-card rounded-2xl p-6 shadow-sm text-center border-t-4 border-blue-600">
                        <span class="bg-emerald-100 text-emerald-800 text-[11px] px-3 py-1 rounded-full font-bold inline-block mb-3 border border-emerald-200 shadow-sm">
                            🔴 LIVE ONLINE
                        </span>
                        
                        <div class="bg-slate-50 rounded-xl p-3 mb-4 border border-slate-200">
                            <h3 class="font-bold text-slate-800 text-xl" id="live-class-name">ห้องเรียน</h3>
                            <p class="text-sm font-semibold text-blue-600 mt-1" id="live-subject-name">วิชา: -</p>
                        </div>
                        
                        <div class="bg-white p-4 rounded-2xl shadow-inner inline-block border border-slate-200">
                            <div id="qrcode" class="flex justify-center"></div>
                        </div>
                        <p class="text-xs text-slate-500 mt-3 font-medium">ให้นักเรียนสแกน QR Code เพื่อเลือกชื่อและขอ OTP</p>
                        
                        <div class="mt-4 pt-4 border-t border-slate-100">
                            <input type="text" id="session-link-input" readonly class="w-full text-xs bg-slate-100 border border-slate-200 rounded-lg p-2.5 text-slate-600 font-mono text-center mb-2 focus:outline-none select-all">
                            <button onclick="copySessionLink()" class="w-full text-xs bg-blue-50 hover:bg-blue-100 text-blue-600 font-semibold py-2.5 px-3 rounded-xl border border-blue-200 transition flex items-center justify-center space-x-1.5 active:scale-95">
                                <i class="fa-solid fa-copy"></i>
                                <span>คัดลอกลิงก์ให้นักเรียน</span>
                            </button>
                        </div>
                    </div>

                    <div class="glass-card rounded-2xl p-6 shadow-md bg-slate-900 text-white">
                        <h4 class="font-bold text-base mb-1 flex items-center">
                            <i class="fa-solid fa-shield-halved text-amber-400 mr-2"></i> กรอก OTP ยืนยันให้เด็ก
                        </h4>
                        <p class="text-xs text-slate-400 mb-4">พิมพ์รหัส 6 หลักที่นักเรียนแจ้งเพื่อยืนยันเข้าเรียนทันที</p>
                        <div class="flex space-x-2">
                            <input type="text" id="teacher-otp-input" placeholder="000000" maxlength="6" class="w-full text-center tracking-widest font-mono text-2xl font-bold bg-slate-800 border border-slate-700 rounded-xl p-2.5 text-amber-400 focus:outline-none focus:ring-2 focus:ring-amber-400">
                            <button onclick="verifyTeacherOTP()" class="bg-amber-500 hover:bg-amber-600 active:scale-95 text-slate-950 font-bold px-5 rounded-xl transition shadow-lg flex items-center justify-center shrink-0">
                                ยืนยัน
                            </button>
                        </div>
                    </div>
                </div>

                <div class="lg:col-span-2">
                    <div class="glass-card rounded-2xl p-6 shadow-sm min-h-full flex flex-col justify-between">
                        <div>
                            <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-2">
                                <div>
                                    <h3 class="font-bold text-lg text-slate-800 flex items-center">
                                        ตารางเช็คชื่อนักเรียน Real-time
                                        <span class="ml-2 w-2.5 h-2.5 rounded-full bg-emerald-500 pulse-slow"></span>
                                    </h3>
                                </div>
                                <div class="flex space-x-2 text-xs font-semibold">
                                    <span class="px-3 py-1.5 rounded-lg bg-red-50 text-red-700 border border-red-200">🔴 ยังไม่เช็ค: <span id="cnt-absent">0</span></span>
                                    <span class="px-3 py-1.5 rounded-lg bg-amber-50 text-amber-800 border border-amber-200">🟡 รออนุมัติ: <span id="cnt-pending">0</span></span>
                                    <span class="px-3 py-1.5 rounded-lg bg-emerald-50 text-emerald-800 border border-emerald-200">🟢 เข้าเรียน: <span id="cnt-present">0</span></span>
                                </div>
                            </div>

                            <div class="overflow-x-auto rounded-xl border border-slate-200 shadow-sm">
                                <table class="w-full text-left border-collapse text-sm">
                                    <thead class="bg-slate-100 text-slate-700 font-bold text-xs uppercase tracking-wider">
                                        <tr>
                                            <th class="p-3.5 border-b text-center">เลขที่</th>
                                            <th class="p-3.5 border-b text-center">รหัสนักเรียน</th>
                                            <th class="p-3.5 border-b">ชื่อ - นามสกุล</th>
                                            <th class="p-3.5 border-b text-center">สถานะ</th>
                                            <th class="p-3.5 border-b text-center">OTP</th>
                                            <th class="p-3.5 border-b text-center">จัดการ</th>
                                        </tr>
                                    </thead>
                                    <tbody id="live-students-tbody" class="divide-y divide-slate-100 bg-white">
                                        <tr>
                                            <td colspan="6" class="text-center py-8 text-slate-400">กรุณาเลือกห้องเรียนและเปิดคาบเรียน...</td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- MANAGEMENTS TABS -->
        <div id="tab-classes" class="tab-content hidden space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <h3 class="font-bold text-lg text-slate-800 mb-4 flex items-center">
                    <i class="fa-solid fa-school text-blue-600 mr-2"></i> จัดการห้องเรียน
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-3 mb-6">
                    <input type="text" id="new-class-name" placeholder="ชื่อห้องเรียน (เช่น ม.4/1)" class="bg-slate-50 border border-slate-300 rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none">
                    <input type="text" id="new-class-desc" placeholder="คำอธิบาย (เช่น สายวิทย์-คณิต)" class="bg-slate-50 border border-slate-300 rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none">
                    <button onclick="addClass()" class="bg-blue-600 hover:bg-blue-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow-md transition text-sm">
                        <i class="fa-solid fa-plus mr-1"></i> เพิ่มห้องเรียน
                    </button>
                </div>
                <div class="overflow-x-auto rounded-xl border border-slate-200 shadow-sm">
                    <table class="w-full text-left border-collapse text-sm">
                        <thead class="bg-slate-100 text-slate-700 font-bold text-xs uppercase">
                            <tr>
                                <th class="p-3.5 border-b">ชื่อห้องเรียน</th>
                                <th class="p-3.5 border-b">รายละเอียด</th>
                                <th class="p-3.5 border-b text-center">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="classes-list-tbody" class="divide-y divide-slate-100 bg-white"></tbody>
                    </table>
                </div>
            </div>
        </div>

        <div id="tab-students" class="tab-content hidden space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <h3 class="font-bold text-lg text-slate-800 mb-4 flex items-center">
                    <i class="fa-solid fa-users text-blue-600 mr-2"></i> จัดการนักเรียน
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-5 gap-3 mb-6">
                    <select id="manage-student-class-select" class="bg-slate-50 border border-slate-300 rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                        <option value="">-- เลือกห้องเรียน --</option>
                    </select>
                    <input type="number" id="new-std-no" placeholder="เลขที่" class="bg-slate-50 border border-slate-300 rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none">
                    <input type="text" id="new-std-code" placeholder="รหัสนักเรียน" class="bg-slate-50 border border-slate-300 rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none">
                    <input type="text" id="new-std-name" placeholder="ชื่อ-นามสกุล" class="bg-slate-50 border border-slate-300 rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none">
                    <button onclick="addStudent()" class="bg-blue-600 hover:bg-blue-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow-md transition text-sm">
                        <i class="fa-solid fa-user-plus mr-1"></i> เพิ่มนักเรียน
                    </button>
                </div>
                <div class="overflow-x-auto rounded-xl border border-slate-200 shadow-sm">
                    <table class="w-full text-left border-collapse text-sm">
                        <thead class="bg-slate-100 text-slate-700 font-bold text-xs uppercase">
                            <tr>
                                <th class="p-3.5 border-b">เลขที่</th>
                                <th class="p-3.5 border-b">รหัส</th>
                                <th class="p-3.5 border-b">ชื่อ-นามสกุล</th>
                                <th class="p-3.5 border-b">ห้องเรียน</th>
                                <th class="p-3.5 border-b text-center">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="students-list-tbody" class="divide-y divide-slate-100 bg-white"></tbody>
                    </table>
                </div>
            </div>
        </div>

        <div id="tab-reports" class="tab-content hidden space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <h3 class="font-bold text-lg text-slate-800 mb-4 flex items-center">
                    <i class="fa-solid fa-chart-line text-blue-600 mr-2"></i> รายงานสถิติและประวัติการเช็คชื่อ
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
                    <div class="bg-blue-50 border border-blue-100 p-4 rounded-xl">
                        <p class="text-xs font-semibold text-blue-600 uppercase">ห้องเรียนทั้งหมด</p>
                        <p class="text-2xl font-bold text-blue-900 mt-1" id="stat-total-classes">0</p>
                    </div>
                    <div class="bg-indigo-50 border border-indigo-100 p-4 rounded-xl">
                        <p class="text-xs font-semibold text-indigo-600 uppercase">นักเรียนทั้งหมด</p>
                        <p class="text-2xl font-bold text-indigo-900 mt-1" id="stat-total-students">0</p>
                    </div>
                    <div class="bg-emerald-50 border border-emerald-100 p-4 rounded-xl">
                        <p class="text-xs font-semibold text-emerald-600 uppercase">จำนวนการเปิดคาบเรียน</p>
                        <p class="text-2xl font-bold text-emerald-900 mt-1" id="stat-total-history">0</p>
                    </div>
                </div>
                <div class="overflow-x-auto rounded-xl border border-slate-200 shadow-sm">
                    <table class="w-full text-left border-collapse text-sm">
                        <thead class="bg-slate-100 text-slate-700 font-bold text-xs uppercase">
                            <tr>
                                <th class="p-3.5 border-b">วัน-เวลา</th>
                                <th class="p-3.5 border-b">ห้องเรียน</th>
                                <th class="p-3.5 border-b">วิชา</th>
                                <th class="p-3.5 border-b text-center">ผู้เข้าเรียน</th>
                            </tr>
                        </thead>
                        <tbody id="history-list-tbody" class="divide-y divide-slate-100 bg-white"></tbody>
                    </table>
                </div>
            </div>
        </div>

    </main>

    <!-- JS LOGIC DIRECT SYNC FIXED -->
    <script>
        const CLOUD_DB_BASE_URL = "https://checkin-realtime-default-rtdb.asia-southeast1.firebasedatabase.app";

        let pollingInterval = null;
        let currentSessionId = null;
        let currentSessionData = null;
        let currentStudentSelectedId = null;

        const defaultClasses = [{ id: 'c1', name: 'ม.4/1', desc: 'สายวิทยาศาสตร์-คณิตศาสตร์' }];
        const defaultStudents = [
            { id: 's1', classId: 'c1', no: 1, stdId: '10001', name: 'นายกิตติพงษ์ วงศ์สว่าง' },
            { id: 's2', classId: 'c1', no: 2, stdId: '10002', name: 'นางสาวจิราพร แสงทอง' },
            { id: 's3', classId: 'c1', no: 3, stdId: '10003', name: 'นายณัฐพงษ์ ใจดี' }
        ];

        window.appState = {
            classes: JSON.parse(localStorage.getItem('sc_classes')) || defaultClasses,
            students: JSON.parse(localStorage.getItem('sc_students')) || defaultStudents,
            history: JSON.parse(localStorage.getItem('sc_history')) || []
        };

        if (!localStorage.getItem('sc_classes')) saveState();

        function saveState() {
            localStorage.setItem('sc_classes', JSON.stringify(window.appState.classes));
            localStorage.setItem('sc_students', JSON.stringify(window.appState.students));
            localStorage.setItem('sc_history', JSON.stringify(window.appState.history));
        }

        function getSessionIdFromUrl() {
            const urlParams = new URLSearchParams(window.location.search);
            let sessionId = urlParams.get('session');
            if (!sessionId && window.location.hash) {
                const hashParams = new URLSearchParams(window.location.hash.replace('#', '?'));
                sessionId = hashParams.get('session');
            }
            return sessionId;
        }

        window.addEventListener('DOMContentLoaded', () => {
            const sessionIdParam = getSessionIdFromUrl();

            if (sessionIdParam) {
                currentSessionId = sessionIdParam;
                document.getElementById('main-header').classList.add('hidden');
                document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
                document.getElementById('student-view').classList.remove('hidden');
                
                fetchStudentSessionData();
                pollingInterval = setInterval(fetchStudentSessionData, 1500);
            } else {
                populateClassDropdowns();
                renderClassesList();
                renderStudentsList();
                renderReports();
                switchTab('dashboard');
            }
        });

        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            const target = document.getElementById(`tab-${tabId}`);
            if (target) target.classList.remove('hidden');

            document.querySelectorAll('.nav-btn').forEach(btn => {
                btn.classList.remove('text-blue-400');
                btn.classList.add('text-slate-300');
            });
            const activeNav = document.getElementById(`nav-${tabId}`);
            if (activeNav) {
                activeNav.classList.add('text-blue-400');
                activeNav.classList.remove('text-slate-300');
            }
        }

        function populateClassDropdowns() {
            const selectTeacher = document.getElementById('teacher-class-select');
            const selectManage = document.getElementById('manage-student-class-select');
            const optionsHtml = window.appState.classes.map(c => `<option value="${c.id}">${c.name}</option>`).join('');
            
            if (selectTeacher) selectTeacher.innerHTML = '<option value="">-- เลือกห้องเรียน --</option>' + optionsHtml;
            if (selectManage) selectManage.innerHTML = '<option value="">-- เลือกห้องเรียน --</option>' + optionsHtml;
        }

        async function toggleClassSession() {
            const btn = document.getElementById('btn-toggle-session');
            const classSelect = document.getElementById('teacher-class-select');
            const subjectInput = document.getElementById('teacher-subject-input');

            if (!currentSessionId) {
                const classId = classSelect.value;
                const subject = subjectInput.value.trim() || 'เช็คชื่อเข้าเรียน';
                if (!classId) {
                    Swal.fire({ icon: 'warning', title: 'โปรดเลือกห้องเรียนก่อนเริ่มคาบเรียน' });
                    return;
                }

                const selectedClass = window.appState.classes.find(c => c.id === classId);
                const classStudents = window.appState.students.filter(s => s.classId === classId);

                if (classStudents.length === 0) {
                    Swal.fire({ icon: 'error', title: 'ไม่พบนักเรียนในห้องเรียนนี้', text: 'โปรดเพิ่มนักเรียนในเมนูก่อนครับ' });
                    return;
                }

                currentSessionId = 'S_' + Date.now();
                
                // สร้าง Object นักเรียนและส่งขึ้น คลาวด์ ทันที
                const sessionStudentsMap = {};
                classStudents.forEach(s => {
                    sessionStudentsMap[s.id] = {
                        id: s.id,
                        no: s.no,
                        stdId: s.stdId,
                        name: s.name,
                        status: 'absent',
                        otp: ''
                    };
                });

                const payload = {
                    sessionId: currentSessionId,
                    classId: selectedClass.id,
                    className: selectedClass.name,
                    subject: subject,
                    active: true,
                    students: sessionStudentsMap
                };

                currentSessionData = payload;
                renderTeacherLiveTable(payload);

                try {
                    await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}.json`, {
                        method: 'PUT',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });
                } catch (e) {
                    console.error("Cloud push failed", e);
                }

                btn.className = "w-full bg-rose-600 hover:bg-rose-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow-md transition duration-200 flex items-center justify-center space-x-2 text-sm";
                btn.innerHTML = `<i class="fa-solid fa-stop"></i><span>🔴 ปิดคาบเรียน</span>`;
                classSelect.disabled = true;
                subjectInput.disabled = true;

                document.getElementById('active-session-container').classList.remove('hidden');
                document.getElementById('live-class-name').textContent = selectedClass.name;
                document.getElementById('live-subject-name').textContent = `วิชา: ${subject}`;

                const baseUrl = window.location.origin + window.location.pathname;
                const studentUrl = `${baseUrl}?session=${currentSessionId}`;
                document.getElementById('session-link-input').value = studentUrl;
                
                document.getElementById('qrcode').innerHTML = '';
                new QRCode(document.getElementById("qrcode"), { text: studentUrl, width: 160, height: 160 });

                pollingInterval = setInterval(fetchTeacherSessionData, 1500);
                Swal.fire({ icon: 'success', title: 'เปิดคาบเรียนสำเร็จ!', timer: 1200, showConfirmButton: false });

            } else {
                clearInterval(pollingInterval);
                try {
                    await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}.json`, { method: 'DELETE' });
                } catch (e) {}

                currentSessionId = null;
                currentSessionData = null;
                btn.className = "w-full bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow-md transition duration-200 flex items-center justify-center space-x-2 text-sm";
                btn.innerHTML = `<i class="fa-solid fa-play"></i><span>🟢 เริ่มเปิดคาบเรียน (Real-time)</span>`;
                classSelect.disabled = false;
                subjectInput.disabled = false;
                document.getElementById('active-session-container').classList.add('hidden');
                Swal.fire({ icon: 'info', title: 'ปิดคาบเรียนแล้ว', timer: 1200, showConfirmButton: false });
            }
        }

        async function fetchTeacherSessionData() {
            if (!currentSessionId) return;
            try {
                const res = await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}.json`);
                const data = await res.json();
                if (data) {
                    currentSessionData = data;
                    renderTeacherLiveTable(data);
                }
            } catch (e) {}
        }

        function renderTeacherLiveTable(sessionData) {
            const tbody = document.getElementById('live-students-tbody');
            if (!sessionData || !sessionData.students) return;

            let studentsList = Array.isArray(sessionData.students) 
                ? sessionData.students.filter(Boolean) 
                : Object.values(sessionData.students);
                
            studentsList.sort((a, b) => (a.no || 0) - (b.no || 0));

            let presentCount = 0, pendingCount = 0, absentCount = 0;

            tbody.innerHTML = studentsList.map(s => {
                let statusBadge = '', actionBtn = '';
                if (s.status === 'present') {
                    presentCount++;
                    statusBadge = `<span class="bg-emerald-100 text-emerald-800 text-xs px-2.5 py-1 rounded-full font-bold">เข้าเรียน</span>`;
                } else if (s.status === 'pending') {
                    pendingCount++;
                    statusBadge = `<span class="bg-amber-100 text-amber-800 text-xs px-2.5 py-1 rounded-full font-bold pulse-slow">รออนุมัติ</span>`;
                    actionBtn = `<button onclick="approveStudent('${s.id}')" class="bg-emerald-600 text-white text-xs px-3 py-1 rounded-lg hover:bg-emerald-700 transition">อนุมัติ</button>`;
                } else {
                    absentCount++;
                    statusBadge = `<span class="bg-slate-100 text-slate-500 text-xs px-2.5 py-1 rounded-full">ยังไม่เช็ค</span>`;
                }

                return `
                    <tr class="hover:bg-slate-50">
                        <td class="p-3.5 font-semibold text-center">${s.no || '-'}</td>
                        <td class="p-3.5 font-mono text-xs text-center">${s.stdId || '-'}</td>
                        <td class="p-3.5 font-medium">${s.name || 'ไม่ระบุชื่อ'}</td>
                        <td class="p-3.5 text-center">${statusBadge}</td>
                        <td class="p-3.5 text-center font-mono font-bold text-amber-600">${s.otp || '-'}</td>
                        <td class="p-3.5 text-center">${actionBtn}</td>
                    </tr>
                `;
            }).join('');

            document.getElementById('cnt-present').textContent = presentCount;
            document.getElementById('cnt-pending').textContent = pendingCount;
            document.getElementById('cnt-absent').textContent = absentCount;
        }

        async function approveStudent(stdId) {
            if (!currentSessionId) return;
            if (currentSessionData && currentSessionData.students && currentSessionData.students[stdId]) {
                currentSessionData.students[stdId].status = 'present';
                renderTeacherLiveTable(currentSessionData);
            }
            try {
                await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}/students/${stdId}.json`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ status: 'present' })
                });
            } catch(e){}
        }

        async function verifyTeacherOTP() {
            const input = document.getElementById('teacher-otp-input');
            const otpCode = input.value.trim();
            if (!otpCode || otpCode.length !== 6) return;

            if (!currentSessionData || !currentSessionData.students) return;

            let studentsList = Array.isArray(currentSessionData.students) 
                ? currentSessionData.students.filter(Boolean) 
                : Object.values(currentSessionData.students);

            const student = studentsList.find(s => s.otp === otpCode && s.status === 'pending');

            if (student) {
                await approveStudent(student.id);
                input.value = '';
                Swal.fire({ icon: 'success', title: `ยืนยันคุณ ${student.name} สำเร็จ`, timer: 1200, showConfirmButton: false });
            } else {
                Swal.fire({ icon: 'error', title: 'รหัส OTP ไม่ถูกต้อง' });
            }
        }

        function copySessionLink() {
            const input = document.getElementById('session-link-input');
            input.select();
            document.execCommand('copy');
            Swal.fire({ icon: 'success', title: 'คัดลอกลิงก์เรียบร้อย', timer: 1000, showConfirmButton: false });
        }

        // --- STUDENT FETCH FIX ---
        async function fetchStudentSessionData() {
            const loadingState = document.getElementById('student-state-loading');
            const closedState = document.getElementById('student-state-closed');
            const activeState = document.getElementById('student-state-active');

            if (!currentSessionId) currentSessionId = getSessionIdFromUrl();

            try {
                const res = await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}.json`);
                const data = await res.json();

                if (!data) {
                    loadingState.classList.add('hidden');
                    activeState.classList.add('hidden');
                    closedState.classList.remove('hidden');
                    return;
                }

                loadingState.classList.add('hidden');
                closedState.classList.add('hidden');
                activeState.classList.remove('hidden');

                document.getElementById('student-class-title').textContent = `ห้อง ${data.className || ''}`;
                document.getElementById('student-session-info').textContent = `วิชา: ${data.subject || ''}`;

                const dropdown = document.getElementById('student-dropdown');
                const previousVal = dropdown.value;
                
                let studentsData = data.students || {};
                let studentsList = [];

                if (Array.isArray(studentsData)) {
                    studentsList = studentsData.filter(Boolean);
                } else if (typeof studentsData === 'object') {
                    studentsList = Object.values(studentsData);
                }

                // Fallback ถ้ารายชื่อจากคลาวด์ยังไม่มา ให้ใช้รายชื่อ local
                if (studentsList.length === 0 && window.appState.students) {
                    studentsList = window.appState.students;
                }

                studentsList.sort((a,b) => (a.no || 0) - (b.no || 0));

                if (studentsList.length > 0) {
                    dropdown.innerHTML = '<option value="">-- เลือกชื่อของคุณ --</option>' + 
                        studentsList.map(s => `<option value="${s.id}">${s.no}. ${s.name} (${s.stdId || ''})</option>`).join('');
                    if (previousVal) dropdown.value = previousVal;
                }

                if (currentStudentSelectedId && studentsData[currentStudentSelectedId]) {
                    const myState = studentsData[currentStudentSelectedId];
                    if (myState.status === 'present') {
                        document.getElementById('student-step-select').classList.add('hidden');
                        document.getElementById('student-step-otp').classList.add('hidden');
                        document.getElementById('student-step-success').classList.remove('hidden');
                    } else if (myState.status === 'pending') {
                        document.getElementById('student-step-select').classList.add('hidden');
                        document.getElementById('student-step-otp').classList.remove('hidden');
                        document.getElementById('display-otp-code').textContent = myState.otp;
                    }
                }
            } catch (e) {
                loadingState.classList.add('hidden');
                closedState.classList.remove('hidden');
            }
        }

        async function requestStudentOTP() {
            const dropdown = document.getElementById('student-dropdown');
            const selectedId = dropdown.value;

            if (!selectedId) {
                Swal.fire({ icon: 'warning', title: 'โปรดเลือกชื่อของคุณ' });
                return;
            }

            const otp = Math.floor(100000 + Math.random() * 900000).toString();
            currentStudentSelectedId = selectedId;

            try {
                await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}/students/${selectedId}.json`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ status: 'pending', otp: otp })
                });

                document.getElementById('student-step-select').classList.add('hidden');
                document.getElementById('student-step-otp').classList.remove('hidden');
                document.getElementById('display-otp-code').textContent = otp;
            } catch (e) {
                Swal.fire({ icon: 'error', title: 'เกิดข้อผิดพลาดในการขอ OTP' });
            }
        }

        // --- MANAGEMENT ---
        function addClass() {
            const name = document.getElementById('new-class-name').value.trim();
            const desc = document.getElementById('new-class-desc').value.trim();
            if (!name) return;
            const id = 'c_' + Date.now();
            window.appState.classes.push({ id, name, desc });
            saveState();
            populateClassDropdowns();
            renderClassesList();
            document.getElementById('new-class-name').value = '';
            document.getElementById('new-class-desc').value = '';
        }

        function deleteClass(id) {
            window.appState.classes = window.appState.classes.filter(c => c.id !== id);
            window.appState.students = window.appState.students.filter(s => s.classId !== id);
            saveState();
            populateClassDropdowns();
            renderClassesList();
            renderStudentsList();
        }

        function renderClassesList() {
            const tbody = document.getElementById('classes-list-tbody');
            if (!tbody) return;
            tbody.innerHTML = window.appState.classes.map(c => `
                <tr class="hover:bg-slate-50">
                    <td class="p-3.5 font-semibold text-slate-800">${c.name}</td>
                    <td class="p-3.5 text-slate-500">${c.desc || '-'}</td>
                    <td class="p-3.5 text-center">
                        <button onclick="deleteClass('${c.id}')" class="text-rose-600 hover:text-rose-800 font-semibold px-2 py-1"><i class="fa-solid fa-trash"></i></button>
                    </td>
                </tr>
            `).join('');
            document.getElementById('stat-total-classes').textContent = window.appState.classes.length;
        }

        function addStudent() {
            const classId = document.getElementById('manage-student-class-select').value;
            const no = parseInt(document.getElementById('new-std-no').value);
            const stdId = document.getElementById('new-std-code').value.trim();
            const name = document.getElementById('new-std-name').value.trim();

            if (!classId || !no || !stdId || !name) {
                Swal.fire({ icon: 'warning', title: 'กรอกข้อมูลให้ครบถ้วน' });
                return;
            }

            const id = 's_' + Date.now();
            window.appState.students.push({ id, classId, no, stdId, name });
            saveState();
            renderStudentsList();

            document.getElementById('new-std-no').value = '';
            document.getElementById('new-std-code').value = '';
            document.getElementById('new-std-name').value = '';
        }

        function deleteStudent(id) {
            window.appState.students = window.appState.students.filter(s => s.id !== id);
            saveState();
            renderStudentsList();
        }

        function renderStudentsList() {
            const tbody = document.getElementById('students-list-tbody');
            if (!tbody) return;
            tbody.innerHTML = window.appState.students.map(s => {
                const cls = window.appState.classes.find(c => c.id === s.classId);
                return `
                    <tr class="hover:bg-slate-50">
                        <td class="p-3.5 font-semibold">${s.no}</td>
                        <td class="p-3.5 font-mono text-xs">${s.stdId}</td>
                        <td class="p-3.5 font-medium">${s.name}</td>
                        <td class="p-3.5 text-slate-500">${cls ? cls.name : '-'}</td>
                        <td class="p-3.5 text-center">
                            <button onclick="deleteStudent('${s.id}')" class="text-rose-600 hover:text-rose-800 font-semibold px-2 py-1"><i class="fa-solid fa-trash"></i></button>
                        </td>
                    </tr>
                `;
            }).join('');
            document.getElementById('stat-total-students').textContent = window.appState.students.length;
        }

        function renderReports() {
            const tbody = document.getElementById('history-list-tbody');
            if (!tbody) return;
            tbody.innerHTML = window.appState.history.map(h => `
                <tr class="hover:bg-slate-50">
                    <td class="p-3.5 font-mono text-xs text-slate-600">${h.timestamp}</td>
                    <td class="p-3.5 font-semibold">${h.className}</td>
                    <td class="p-3.5">${h.subject}</td>
                    <td class="p-3.5 text-center font-bold text-emerald-600">${h.stats}</td>
                </tr>
            `).join('');
            document.getElementById('stat-total-history').textContent = window.appState.history.length;
        }
    </script>
</body>
</html>
