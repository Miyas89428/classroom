<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>課堂管理系統</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;600;700&display=swap');
        body {
            font-family: 'Noto Sans TC', sans-serif;
        }
        .tab-content {
            display: none;
        }
        .tab-content.active {
            display: block;
        }
        .password-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen">
    <!-- 密碼輸入覆蓋層 -->
    <div id="passwordOverlay" class="password-overlay" style="display: none;">
        <div class="bg-white rounded-2xl shadow-2xl p-8 max-w-md w-full mx-4">
            <div class="text-center mb-6">
                <div class="text-4xl mb-4">🔐</div>
                <h2 class="text-2xl font-bold text-gray-800 mb-2">權限驗證</h2>
                <p class="text-gray-600" id="passwordPromptText">請輸入密碼以進入教師觀察記錄</p>
            </div>
            
            <div class="space-y-4">
                <input type="password" id="passwordInput" placeholder="請輸入密碼" 
                       class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200"
                       onkeypress="if(event.key==='Enter') checkPassword()">
                
                <div class="flex gap-3">
                    <button onclick="checkPassword()" 
                            class="flex-1 px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors duration-200 font-semibold">
                        🔓 確認
                    </button>
                    <button onclick="closePasswordOverlay()" 
                            class="flex-1 px-6 py-3 bg-gray-500 text-white rounded-lg hover:bg-gray-600 transition-colors duration-200 font-semibold">
                        ❌ 取消
                    </button>
                </div>
                
                <div id="passwordError" class="text-red-500 text-sm text-center" style="display: none;">
                    密碼錯誤，請重新輸入
                </div>
            </div>
        </div>
    </div>

    <div class="container mx-auto px-4 py-8">
        <!-- 標題 -->
        <div class="text-center mb-8">
            <div class="flex items-center justify-center mb-4">
                <div id="schoolLogoContainer" class="mr-4" style="display: none;">
                    <img id="schoolLogoImage" src="" alt="學校標誌" 
                         class="w-16 h-16 object-contain border-2 border-gray-300 rounded-lg shadow-md">
                </div>
                <h1 class="text-4xl font-bold text-gray-800">新北市二重國小資優班</h1>
            </div>
        </div>

        <!-- 標籤切換 -->
        <div class="max-w-6xl mx-auto mb-6">
            <div class="flex bg-white rounded-lg shadow-md p-1">
                <button onclick="switchTab('attendance')" id="attendance-tab" 
                        class="flex-1 px-4 py-3 rounded-md font-semibold transition-all duration-200 bg-blue-600 text-white text-sm">
                    📝 課堂點名
                </button>
                <button onclick="showPasswordPrompt('observation')" id="observation-tab" 
                        class="flex-1 px-4 py-3 rounded-md font-semibold transition-all duration-200 text-gray-600 hover:bg-gray-100 text-sm">
                    📊 教師觀察記錄
                </button>
                <button onclick="showPasswordPrompt('backend')" id="backend-tab" 
                        class="flex-1 px-4 py-3 rounded-md font-semibold transition-all duration-200 text-gray-600 hover:bg-gray-100 text-sm">
                    ⚙️ 權限控制與設定
                </button>
            </div>
        </div>

        <!-- 課堂點名區域 -->
        <div id="attendance-content" class="tab-content active">
            <div class="max-w-4xl mx-auto bg-white rounded-2xl shadow-xl p-8">
                <form id="attendanceForm" class="space-y-6">
                    <!-- 課程基本資訊 -->
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <div class="space-y-2">
                            <label for="attendanceTeacher" class="block text-sm font-semibold text-gray-700">👨‍🏫 授課教師</label>
                            <select id="attendanceTeacher" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇授課教師</option>
                                <option value="林信加">林信加 教師</option>
                                <option value="曾敏雅">曾敏雅 教師</option>
                            </select>
                        </div>

                        <div class="space-y-2">
                            <label for="attendanceGrade" class="block text-sm font-semibold text-gray-700">🎓 年級</label>
                            <select id="attendanceGrade" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇年級</option>
                                <option value="三年級">三年級</option>
                                <option value="四年級">四年級</option>
                                <option value="五年級">五年級</option>
                                <option value="六年級">六年級</option>
                            </select>
                        </div>

                        <div class="space-y-2">
                            <label for="attendanceCourse" class="block text-sm font-semibold text-gray-700">📚 課程組別</label>
                            <select id="attendanceCourse" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇課程組別</option>
                                <option value="語文A">語文A</option>
                                <option value="語文B">語文B</option>
                                <option value="數理A">數理A</option>
                                <option value="創造力A">創造力A</option>
                                <option value="創造力B">創造力B</option>
                                <option value="情意">情意</option>
                                <option value="獨立研究A">獨立研究A</option>
                                <option value="獨立研究B">獨立研究B</option>
                                <option value="獨立研究C">獨立研究C</option>
                                <option value="領導力">領導力</option>
                            </select>
                        </div>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div class="space-y-2">
                            <label for="attendanceWeekDate" class="block text-sm font-semibold text-gray-700">📅 周次/日期/星期</label>
                            <select id="attendanceWeekDate" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇周次/日期/星期</option>
                            </select>
                        </div>

                        <div class="space-y-2">
                            <label for="attendancePeriod" class="block text-sm font-semibold text-gray-700">⏰ 上課節次</label>
                            <select id="attendancePeriod" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇節次</option>
                                <option value="早自習">早自習 (07:50-08:30)</option>
                                <option value="第一節">第一節 (08:40-09:20)</option>
                                <option value="第二節">第二節 (09:30-10:10)</option>
                                <option value="第三節">第三節 (10:30-11:10)</option>
                                <option value="第四節">第四節 (11:20-12:00)</option>
                                <option value="午休">午休 (12:40-13:20)</option>
                                <option value="第五節">第五節 (13:30-14:10)</option>
                                <option value="第六節">第六節 (14:20-15:00)</option>
                                <option value="第七節">第七節 (15:20-16:00)</option>
                            </select>
                        </div>
                    </div>

                    <!-- 學生姓名與出席狀態 -->
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <div class="space-y-2">
                            <label for="attendanceStudentName" class="block text-sm font-semibold text-gray-700">👤 學生姓名</label>
                            <input type="text" id="attendanceStudentName" placeholder="請輸入學生姓名" 
                                   class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200">
                        </div>
                        
                        <div class="md:col-span-2 space-y-2">
                            <label class="block text-sm font-semibold text-gray-700">📝 出席狀態</label>
                            <div class="space-y-2">
                                <label class="flex items-center space-x-2">
                                    <input type="radio" name="attendanceStatus" value="出席" checked
                                           class="text-green-600 focus:ring-green-500">
                                    <span class="text-sm text-gray-700">✅ 出席</span>
                                </label>

                                <div class="flex items-center space-x-2">
                                    <input type="radio" name="attendanceStatus" value="遲到" onchange="toggleLateReason()"
                                           class="text-yellow-600 focus:ring-yellow-500">
                                    <span class="text-sm text-gray-700">⏰ 遲到，原因：</span>
                                    <input type="text" id="lateReason" placeholder="請輸入遲到原因" disabled
                                           class="flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 disabled:bg-gray-100">
                                </div>

                                <div class="flex items-center space-x-2">
                                    <input type="radio" name="attendanceStatus" value="請假" onchange="toggleLeaveReason()"
                                           class="text-red-600 focus:ring-red-500">
                                    <span class="text-sm text-gray-700">🏠 請假，原因：</span>
                                    <input type="text" id="leaveReason" placeholder="請輸入請假原因" disabled
                                           class="flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 disabled:bg-gray-100">
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- 提交按鈕 -->
                    <div class="flex gap-4">
                        <button type="button" onclick="submitAttendance()" 
                                class="flex-1 px-6 py-3 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors duration-200 font-semibold">
                            ✅ 確認點名
                        </button>
                        <button type="button" onclick="saveAttendanceRecords()" 
                                class="px-6 py-3 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors duration-200 font-semibold">
                            💾 儲存記錄
                        </button>
                    </div>
                </form>

                <!-- 點名記錄顯示區域 -->
                <div class="mt-8">
                    <h3 class="text-lg font-semibold text-gray-800 mb-4">📋 課堂點名紀錄表</h3>
                    <div id="attendanceRecordsList" class="space-y-3 max-h-60 overflow-y-auto">
                        <!-- 點名記錄會動態顯示在這裡 -->
                    </div>
                </div>
            </div>
        </div>

        <!-- 教師觀察記錄區域 -->
        <div id="observation-content" class="tab-content">
            <div class="max-w-4xl mx-auto bg-white rounded-2xl shadow-xl p-8">
                <h2 class="text-2xl font-bold text-gray-800 mb-6">教師觀察記錄</h2>
                
                <form id="observationForm" class="space-y-6">
                    <!-- 課程基本資訊 -->
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <div class="space-y-2">
                            <label for="observationTeacher" class="block text-sm font-semibold text-gray-700">👨‍🏫 授課教師</label>
                            <select id="observationTeacher" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇授課教師</option>
                                <option value="林信加">林信加 教師</option>
                                <option value="曾敏雅">曾敏雅 教師</option>
                            </select>
                        </div>

                        <div class="space-y-2">
                            <label for="observationGrade" class="block text-sm font-semibold text-gray-700">🎓 年級</label>
                            <select id="observationGrade" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇年級</option>
                                <option value="三年級">三年級</option>
                                <option value="四年級">四年級</option>
                                <option value="五年級">五年級</option>
                                <option value="六年級">六年級</option>
                            </select>
                        </div>

                        <div class="space-y-2">
                            <label for="course" class="block text-sm font-semibold text-gray-700">📚 課程組別</label>
                            <select id="course" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇課程組別</option>
                                <option value="語文A">語文A</option>
                                <option value="語文B">語文B</option>
                                <option value="數理A">數理A</option>
                                <option value="創造力A">創造力A</option>
                                <option value="創造力B">創造力B</option>
                                <option value="情意">情意</option>
                                <option value="獨立研究A">獨立研究A</option>
                                <option value="獨立研究B">獨立研究B</option>
                                <option value="獨立研究C">獨立研究C</option>
                                <option value="領導力">領導力</option>
                            </select>
                        </div>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div class="space-y-2">
                            <label for="weekDate" class="block text-sm font-semibold text-gray-700">📅 周次/日期/星期</label>
                            <select id="weekDate" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇周次/日期/星期</option>
                            </select>
                        </div>

                        <div class="space-y-2">
                            <label for="period" class="block text-sm font-semibold text-gray-700">⏰ 上課節次</label>
                            <select id="period" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇節次</option>
                                <option value="早自習">早自習 (07:50-08:30)</option>
                                <option value="第一節">第一節 (08:40-09:20)</option>
                                <option value="第二節">第二節 (09:30-10:10)</option>
                                <option value="第三節">第三節 (10:30-11:10)</option>
                                <option value="第四節">第四節 (11:20-12:00)</option>
                                <option value="午休">午休 (12:40-13:20)</option>
                                <option value="第五節">第五節 (13:30-14:10)</option>
                                <option value="第六節">第六節 (14:20-15:00)</option>
                                <option value="第七節">第七節 (15:20-16:00)</option>
                            </select>
                        </div>
                    </div>

                    <!-- 學生姓名 -->
                    <div class="space-y-2">
                        <label for="studentName" class="block text-sm font-semibold text-gray-700">👤 學生姓名</label>
                        <select id="studentName" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                            <option value="">請選擇學生姓名</option>
                        </select>
                    </div>

                    <!-- 課堂觀察記錄 -->
                    <div class="bg-gray-50 p-6 rounded-lg space-y-6">
                        <h3 class="text-lg font-semibold text-gray-800">📊 課堂觀察記錄</h3>
                        
                        <!-- 課堂參與度 -->
                        <div class="space-y-3">
                            <label class="block text-sm font-semibold text-gray-700">🙋‍♂️ 課堂參與度</label>
                            <div id="participationOptions" class="space-y-2">
                                <!-- 動態生成選項 -->
                            </div>
                        </div>

                        <!-- 學習態度 -->
                        <div class="space-y-3">
                            <label class="block text-sm font-semibold text-gray-700">📚 學習態度</label>
                            <div id="attitudeOptions" class="space-y-2">
                                <!-- 動態生成選項 -->
                            </div>
                        </div>

                        <!-- 作業完成情況 -->
                        <div class="space-y-3">
                            <label class="block text-sm font-semibold text-gray-700">📝 作業完成情況</label>
                            <div id="homeworkOptions" class="space-y-2">
                                <!-- 動態生成選項 -->
                            </div>
                        </div>

                        <!-- 行為表現 -->
                        <div class="space-y-2">
                            <label for="behaviorSelect" class="block text-sm font-semibold text-gray-700">🎭 行為表現</label>
                            <select id="behaviorSelect" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <option value="">請選擇行為表現</option>
                            </select>
                        </div>

                        <!-- 特殊事件記錄 -->
                        <div class="space-y-2">
                            <label for="specialEvent" class="block text-sm font-semibold text-gray-700">📋 特殊事件記錄</label>
                            <textarea id="specialEvent" rows="3" placeholder="請描述特殊事件或值得記錄的內容..." 
                                      class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 resize-none"></textarea>
                        </div>
                    </div>

                    <!-- 提交按鈕 -->
                    <div class="flex gap-4">
                        <button type="button" onclick="submitObservation()" 
                                class="flex-1 px-6 py-3 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors duration-200 font-semibold">
                            ✅ 提交觀察記錄
                        </button>
                        <button type="button" onclick="saveObservationRecords()" 
                                class="px-6 py-3 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors duration-200 font-semibold">
                            💾 儲存記錄
                        </button>
                    </div>
                </form>

                <!-- 觀察記錄顯示區域 -->
                <div class="mt-8">
                    <h3 class="text-lg font-semibold text-gray-800 mb-4">📋 課堂觀察紀錄表</h3>
                    <div id="observationRecordsList" class="space-y-3 max-h-60 overflow-y-auto">
                        <!-- 觀察記錄會動態顯示在這裡 -->
                    </div>
                </div>
            </div>
        </div>

        <!-- 權限控制區域 -->
        <div id="backend-content" class="tab-content">
            <div class="max-w-6xl mx-auto space-y-6">
                <!-- 評量標準設定 -->
                <div class="bg-white rounded-2xl shadow-xl p-8">
                    <h2 class="text-2xl font-bold text-gray-800 mb-6">📊 評量標準設定</h2>
                    
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                        <!-- 課堂參與度設定 -->
                        <div class="space-y-4">
                            <h3 class="text-lg font-semibold text-gray-700">🙋‍♂️ 課堂參與度</h3>
                            <div class="flex gap-2">
                                <input type="text" id="newParticipation" placeholder="新增選項" 
                                       class="flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm">
                                <button onclick="addOption('participation')" 
                                        class="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 text-sm">
                                    ➕
                                </button>
                            </div>
                            <div id="participationList" class="space-y-2">
                                <!-- 選項列表 -->
                            </div>
                        </div>

                        <!-- 學習態度設定 -->
                        <div class="space-y-4">
                            <h3 class="text-lg font-semibold text-gray-700">📚 學習態度</h3>
                            <div class="flex gap-2">
                                <input type="text" id="newAttitude" placeholder="新增選項" 
                                       class="flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm">
                                <button onclick="addOption('attitude')" 
                                        class="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 text-sm">
                                    ➕
                                </button>
                            </div>
                            <div id="attitudeList" class="space-y-2">
                                <!-- 選項列表 -->
                            </div>
                        </div>

                        <!-- 作業完成情況設定 -->
                        <div class="space-y-4">
                            <h3 class="text-lg font-semibold text-gray-700">📝 作業完成情況</h3>
                            <div class="flex gap-2">
                                <input type="text" id="newHomework" placeholder="新增選項" 
                                       class="flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm">
                                <button onclick="addOption('homework')" 
                                        class="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 text-sm">
                                    ➕
                                </button>
                            </div>
                            <div id="homeworkList" class="space-y-2">
                                <!-- 選項列表 -->
                            </div>
                        </div>
                    </div>
                    
                    <!-- 儲存設定按鈕 -->
                    <div class="mt-6 text-center">
                        <button onclick="saveEvaluationSettings()" 
                                class="px-8 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors duration-200 font-semibold">
                            💾 儲存評量標準設定
                        </button>
                    </div>
                </div>

                <!-- 行為表現設定 -->
                <div class="bg-white rounded-2xl shadow-xl p-8">
                    <h2 class="text-2xl font-bold text-gray-800 mb-6">🎭 行為表現選項設定</h2>
                    
                    <div class="flex gap-4 mb-6">
                        <input type="text" id="newBehavior" placeholder="輸入行為表現描述" 
                               class="flex-1 px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200">
                        <button onclick="addBehavior()" 
                                class="px-6 py-3 bg-orange-600 text-white rounded-lg hover:bg-orange-700 transition-colors duration-200 font-semibold">
                            ➕ 新增選項
                        </button>
                    </div>

                    <div id="behaviorList" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <!-- 行為選項列表 -->
                    </div>
                    
                    <!-- 儲存設定按鈕 -->
                    <div class="mt-6 text-center">
                        <button onclick="saveBehaviorSettings()" 
                                class="px-8 py-3 bg-orange-600 text-white rounded-lg hover:bg-orange-700 transition-colors duration-200 font-semibold">
                            💾 儲存行為表現設定
                        </button>
                    </div>
                </div>

                <!-- 密碼設定 -->
                <div class="bg-white rounded-2xl shadow-xl p-8">
                    <h2 class="text-2xl font-bold text-gray-800 mb-6">🔐 密碼設定</h2>
                    
                    <div class="space-y-6">
                        <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400">
                            <h3 class="font-semibold text-blue-800 mb-2">🔑 系統密碼管理</h3>
                            <p class="text-sm text-blue-700">設定密碼來保護教師觀察記錄與權限控制功能</p>
                        </div>
                        
                        <div class="space-y-4">
                            <div class="space-y-2">
                                <label for="currentPassword" class="block text-sm font-semibold text-gray-700">目前密碼</label>
                                <input type="password" id="currentPassword" placeholder="請輸入目前密碼" 
                                       class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200">
                            </div>
                            
                            <div class="space-y-2">
                                <label for="newPassword" class="block text-sm font-semibold text-gray-700">新密碼</label>
                                <input type="password" id="newPassword" placeholder="請輸入新密碼" 
                                       class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200">
                            </div>
                            
                            <div class="space-y-2">
                                <label for="confirmPassword" class="block text-sm font-semibold text-gray-700">確認新密碼</label>
                                <input type="password" id="confirmPassword" placeholder="請再次輸入新密碼" 
                                       class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200">
                            </div>
                            
                            <button onclick="changePassword()" 
                                    class="w-full px-6 py-3 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors duration-200 font-semibold">
                                🔄 更新密碼
                            </button>
                        </div>
                        
                        <div class="bg-yellow-50 p-4 rounded-lg border-l-4 border-yellow-400">
                            <h4 class="font-semibold text-yellow-800 mb-2">⚠️ 重要提醒</h4>
                            <ul class="text-sm text-yellow-700 space-y-1">
                                <li>• 目前系統密碼：123</li>
                                <li>• 請妥善保管新密碼，忘記密碼將無法進入受保護功能</li>
                                <li>• 建議使用包含英文字母和數字的組合</li>
                                <li>• 密碼長度建議至少6個字元</li>
                            </ul>
                        </div>
                    </div>
                </div>

                <!-- 學校標誌設定 -->
                <div class="bg-white rounded-2xl shadow-xl p-8">
                    <h2 class="text-2xl font-bold text-gray-800 mb-6">🏫 學校標誌設定</h2>
                    
                    <div class="space-y-6">
                        <!-- 上傳學校標誌 -->
                        <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400">
                            <h3 class="font-semibold text-blue-800 mb-3">📁 上傳學校標誌</h3>
                            <div class="space-y-3">
                                <input type="file" id="schoolLogoFile" accept="image/*" 
                                       class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200 bg-white">
                                <button onclick="uploadSchoolLogo()" 
                                        class="w-full px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors duration-200 font-semibold">
                                    🏫 上傳學校標誌
                                </button>
                            </div>
                        </div>
                        
                        <!-- 標誌預覽 -->
                        <div id="logoPreviewSection" class="bg-gray-50 p-4 rounded-lg" style="display: none;">
                            <h4 class="font-semibold text-gray-700 mb-3">📋 標誌預覽</h4>
                            <div class="flex items-center justify-center">
                                <img id="logoPreviewImage" src="" alt="標誌預覽" 
                                     class="w-16 h-16 object-contain border-2 border-gray-300 rounded-lg shadow-md">
                            </div>
                        </div>
                        
                        <!-- 控制按鈕 -->
                        <div class="flex gap-4">
                            <button onclick="removeSchoolLogo()" 
                                    class="flex-1 px-6 py-3 bg-gray-600 text-white rounded-lg hover:bg-gray-700 transition-colors duration-200 font-semibold">
                                🗑️ 移除標誌
                            </button>
                        </div>
                        
                        <div class="bg-gray-50 p-4 rounded-lg">
                            <h4 class="font-semibold text-gray-700 mb-2">💡 使用說明</h4>
                            <ul class="text-sm text-gray-600 space-y-1">
                                <li>• <strong>建議尺寸：</strong>2x2公分 (約64x64像素)</li>
                                <li>• <strong>支援格式：</strong>jpg, png, gif 等圖片格式</li>
                                <li>• <strong>顯示位置：</strong>標誌會顯示在「新北市二重國小資優班」標題前方</li>
                                <li>• <strong>建議：</strong>使用正方形圖片以獲得最佳顯示效果</li>
                                <li>• <strong>檔案大小：</strong>建議小於2MB</li>
                            </ul>
                        </div>
                    </div>
                </div>

                <!-- 背景圖設定 -->
                <div class="bg-white rounded-2xl shadow-xl p-8">
                    <h2 class="text-2xl font-bold text-gray-800 mb-6">🖼️ 背景圖設定</h2>
                    
                    <div class="space-y-6">
                        <!-- 上傳本地圖片 -->
                        <div class="bg-green-50 p-4 rounded-lg border-l-4 border-green-400">
                            <h3 class="font-semibold text-green-800 mb-3">📁 上傳電腦圖片</h3>
                            <div class="space-y-3">
                                <input type="file" id="backgroundImageFile" accept="image/*" 
                                       class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-transparent transition-all duration-200 bg-white">
                                <button onclick="uploadBackgroundImage()" 
                                        class="w-full px-6 py-3 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors duration-200 font-semibold">
                                    📤 上傳並設定背景圖
                                </button>
                            </div>
                        </div>
                        
                        <!-- 網址設定 -->
                        <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400">
                            <h3 class="font-semibold text-blue-800 mb-3">🌐 使用網址設定</h3>
                            <div class="space-y-3">
                                <input type="url" id="backgroundImageUrl" placeholder="請輸入圖片網址 (例如: https://example.com/image.jpg)" 
                                       class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200">
                                <button onclick="setBackgroundImageFromUrl()" 
                                        class="w-full px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors duration-200 font-semibold">
                                    🖼️ 設定網址背景圖
                                </button>
                            </div>
                        </div>
                        
                        <!-- 透明度控制 -->
                        <div class="space-y-3">
                            <label for="backgroundOpacity" class="block text-sm font-semibold text-gray-700">🎨 背景透明度</label>
                            <input type="range" id="backgroundOpacity" min="0.1" max="1" step="0.1" value="0.3" 
                                   class="w-full" onchange="updateOpacityDisplay()">
                            <div class="text-sm text-gray-600 text-center">
                                透明度: <span id="opacityValue">0.3</span>
                            </div>
                        </div>
                        
                        <!-- 控制按鈕 -->
                        <div class="flex gap-4">
                            <button onclick="applyOpacity()" 
                                    class="flex-1 px-6 py-3 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors duration-200 font-semibold">
                                🎨 套用透明度
                            </button>
                            <button onclick="removeBackgroundImage()" 
                                    class="flex-1 px-6 py-3 bg-gray-600 text-white rounded-lg hover:bg-gray-700 transition-colors duration-200 font-semibold">
                                🗑️ 移除背景圖
                            </button>
                        </div>
                        
                        <div class="bg-gray-50 p-4 rounded-lg">
                            <h4 class="font-semibold text-gray-700 mb-2">💡 使用說明</h4>
                            <ul class="text-sm text-gray-600 space-y-1">
                                <li>• <strong>上傳圖片：</strong>可直接從電腦選擇圖片檔案 (jpg, png, gif 等格式)</li>
                                <li>• <strong>網址設定：</strong>使用網路上的圖片網址</li>
                                <li>• <strong>建議：</strong>使用高解析度圖片以獲得最佳效果</li>
                                <li>• <strong>透明度：</strong>調整背景透明度讓文字更清楚易讀</li>
                                <li>• <strong>套用範圍：</strong>背景圖會套用到整個系統頁面</li>
                            </ul>
                        </div>
                    </div>
                </div>

                <!-- 記錄管理 -->
                <div class="bg-white rounded-2xl shadow-xl p-8">
                    <h2 class="text-2xl font-bold text-gray-800 mb-6">🗂️ 記錄管理</h2>
                    
                    <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400 mb-6">
                        <h3 class="font-semibold text-blue-800 mb-2">📋 記錄管理說明</h3>
                        <ul class="text-sm text-blue-700 space-y-1">
                            <li>• <strong>課程記錄：</strong>以課程組別為主，彙整該課程的點名記錄</li>
                            <li>• <strong>資料整合：</strong>同一堂課的點名記錄會合併為一筆資料</li>
                            <li>• <strong>匯出功能：</strong>可選擇單一課程記錄匯出或匯出所有課程記錄</li>
                            <li>• <strong>記錄來源：</strong>從「課堂點名」功能儲存而來</li>
                        </ul>
                    </div>
                    
                    <div class="flex gap-4 mb-6">
                        <button onclick="exportAllSavedRecords()" 
                                class="px-6 py-3 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors duration-200 font-semibold">
                            📊 匯出所有課程記錄
                        </button>
                        <button onclick="clearAllSavedRecords()" 
                                class="px-6 py-3 bg-red-600 text-white rounded-lg hover:bg-red-700 transition-colors duration-200 font-semibold">
                            🗑️ 清空課程記錄
                        </button>
                        <button onclick="refreshRecordManagement()" 
                                class="px-6 py-3 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors duration-200 font-semibold">
                            🔄 重新整理
                        </button>
                    </div>

                    <div id="recordManagementList" class="space-y-3 max-h-96 overflow-y-auto">
                        <!-- 儲存的記錄會顯示在這裡 -->
                    </div>
                    
                    <!-- 原始記錄區域 -->
                    <div class="mt-8 pt-6 border-t border-gray-200">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">📝 原始記錄（即時資料）</h3>
                        <div class="flex gap-4 mb-4">
                            <button onclick="exportAllToExcel()" 
                                    class="px-4 py-2 bg-gray-600 text-white rounded-lg hover:bg-gray-700 transition-colors duration-200 font-semibold text-sm">
                                📊 匯出原始記錄
                            </button>
                            <button onclick="clearAllRecords()" 
                                    class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition-colors duration-200 font-semibold text-sm">
                                🗑️ 清空原始記錄
                            </button>
                        </div>
                        <div id="allRecordsList" class="space-y-3 max-h-60 overflow-y-auto">
                            <!-- 原始記錄會顯示在這裡 -->
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 全域變數
        let students = ['王小明', '李小華', '張小美', '陳小強', '林小芳']; // 預設學生
        let participationOptions = ['積極參與', '普通參與', '被動參與', '未參與'];
        let attitudeOptions = ['認真專注', '普通表現', '容易分心', '需要提醒'];
        let homeworkOptions = ['完整完成', '部分完成', '未完成', '遲交'];
        let behaviorOptions = ['表現優良', '符合期待', '需要改進', '特殊表現'];
        let attendanceRecords = [];
        let observationRecords = [];
        
        // 密碼設定
        let TEACHER_PASSWORD = '123'; // 可以修改這個密碼
        let currentPasswordTarget = '';
        let isObservationUnlocked = false;
        let isBackendUnlocked = false;

        // 顯示成功訊息
        function showSuccessMessage(message) {
            // 建立訊息元素
            const messageDiv = document.createElement('div');
            messageDiv.className = 'fixed top-4 right-4 bg-green-600 text-white px-6 py-3 rounded-lg shadow-lg z-50 transform transition-all duration-300';
            messageDiv.innerHTML = `
                <div class="flex items-center space-x-2">
                    <span>✅</span>
                    <span>${message}</span>
                </div>
            `;
            
            // 添加到頁面
            document.body.appendChild(messageDiv);
            
            // 5秒後自動移除
            setTimeout(() => {
                messageDiv.style.opacity = '0';
                messageDiv.style.transform = 'translateX(100%)';
                setTimeout(() => {
                    if (messageDiv.parentNode) {
                        messageDiv.parentNode.removeChild(messageDiv);
                    }
                }, 300);
            }, 5000);
        }

        // 控制遲到原因輸入框
        function toggleLateReason() {
            const lateReasonInput = document.getElementById('lateReason');
            const leaveReasonInput = document.getElementById('leaveReason');
            const lateRadio = document.querySelector('input[name="attendanceStatus"][value="遲到"]');
            
            if (lateRadio.checked) {
                lateReasonInput.disabled = false;
                leaveReasonInput.disabled = true;
                leaveReasonInput.value = '';
                lateReasonInput.focus();
            } else {
                lateReasonInput.disabled = true;
                lateReasonInput.value = '';
            }
        }

        // 控制請假原因輸入框
        function toggleLeaveReason() {
            const leaveReasonInput = document.getElementById('leaveReason');
            const lateReasonInput = document.getElementById('lateReason');
            const leaveRadio = document.querySelector('input[name="attendanceStatus"][value="請假"]');
            
            if (leaveRadio.checked) {
                leaveReasonInput.disabled = false;
                lateReasonInput.disabled = true;
                lateReasonInput.value = '';
                leaveReasonInput.focus();
            } else {
                leaveReasonInput.disabled = true;
                leaveReasonInput.value = '';
            }
        }

        // 監聽出席狀態變化
        document.addEventListener('DOMContentLoaded', function() {
            const statusRadios = document.querySelectorAll('input[name="attendanceStatus"]');
            statusRadios.forEach(radio => {
                radio.addEventListener('change', function() {
                    const leaveReasonInput = document.getElementById('leaveReason');
                    const lateReasonInput = document.getElementById('lateReason');
                    
                    // 重置所有輸入框
                    leaveReasonInput.disabled = true;
                    leaveReasonInput.value = '';
                    lateReasonInput.disabled = true;
                    lateReasonInput.value = '';
                    
                    // 根據選擇啟用對應輸入框
                    if (this.value === '請假') {
                        leaveReasonInput.disabled = false;
                        leaveReasonInput.focus();
                    } else if (this.value === '遲到') {
                        lateReasonInput.disabled = false;
                        lateReasonInput.focus();
                    }
                });
            });
        });

        // 初始化
        function init() {
            generateWeekDateOptions();
            updateEvaluationOptions();
            updateBehaviorOptions();
            updateAttendanceRecordsList();
            updateObservationRecordsList();
            updateAllRecordsList();
            updateRecordManagementList();
            
            // 初始化儲存記錄陣列
            if (!window.savedRecords) {
                window.savedRecords = [];
            }
            
            // 監聽點名記錄變化，同步到觀察記錄
            syncObservationFormWithAttendance();
        }

        // 同步觀察記錄表單與點名記錄
        function syncObservationFormWithAttendance() {
            // 如果有點名記錄，自動填入觀察記錄的基本資訊
            if (attendanceRecords.length > 0) {
                const latestRecord = attendanceRecords[attendanceRecords.length - 1];
                
                // 同步基本資訊到觀察記錄表單
                const observationTeacher = document.getElementById('observationTeacher');
                const observationGrade = document.getElementById('observationGrade');
                const observationCourse = document.getElementById('course');
                const observationWeekDate = document.getElementById('weekDate');
                const observationPeriod = document.getElementById('period');
                
                if (observationTeacher) observationTeacher.value = latestRecord.teacher;
                if (observationGrade) observationGrade.value = latestRecord.grade;
                if (observationCourse) observationCourse.value = latestRecord.course;
                if (observationWeekDate) observationWeekDate.value = latestRecord.weekDate;
                if (observationPeriod) observationPeriod.value = latestRecord.period;
                
                // 更新出席學生下拉選單
                updateAttendedStudentsList();
            }
        }

        // 更新出席學生列表
        function updateAttendedStudentsList() {
            const studentSelect = document.getElementById('studentName');
            if (!studentSelect) return;
            
            // 清空現有選項
            studentSelect.innerHTML = '<option value="">請選擇學生姓名</option>';
            
            // 如果沒有點名記錄，顯示提示
            if (attendanceRecords.length === 0) {
                const option = document.createElement('option');
                option.value = '';
                option.textContent = '請先完成課堂點名';
                option.disabled = true;
                studentSelect.appendChild(option);
                return;
            }
            
            // 獲取最新課程的出席學生
            const latestRecord = attendanceRecords[attendanceRecords.length - 1];
            const currentCourseKey = `${latestRecord.course}_${latestRecord.weekDate}_${latestRecord.period}`;
            
            // 篩選出席學生
            const attendedStudents = attendanceRecords
                .filter(record => {
                    const recordKey = `${record.course}_${record.weekDate}_${record.period}`;
                    return recordKey === currentCourseKey && record.attendanceStatus === '出席';
                })
                .map(record => record.studentName);
            
            // 移除重複學生
            const uniqueStudents = [...new Set(attendedStudents)];
            
            if (uniqueStudents.length === 0) {
                const option = document.createElement('option');
                option.value = '';
                option.textContent = '本堂課無出席學生';
                option.disabled = true;
                studentSelect.appendChild(option);
            } else {
                uniqueStudents.forEach(studentName => {
                    const option = document.createElement('option');
                    option.value = studentName;
                    option.textContent = studentName;
                    studentSelect.appendChild(option);
                });
            }
        }

        // 密碼驗證相關函數
        function showPasswordPrompt(target) {
            currentPasswordTarget = target;
            const promptText = target === 'observation' ? '請輸入密碼以進入教師觀察記錄' : '請輸入密碼以進入權限控制';
            document.getElementById('passwordPromptText').textContent = promptText;
            document.getElementById('passwordOverlay').style.display = 'flex';
            document.getElementById('passwordInput').focus();
            document.getElementById('passwordError').style.display = 'none';
        }

        function closePasswordOverlay() {
            document.getElementById('passwordOverlay').style.display = 'none';
            document.getElementById('passwordInput').value = '';
            document.getElementById('passwordError').style.display = 'none';
        }

        function checkPassword() {
            const inputPassword = document.getElementById('passwordInput').value;
            
            if (inputPassword === TEACHER_PASSWORD) {
                if (currentPasswordTarget === 'observation') {
                    isObservationUnlocked = true;
                    switchTab('observation');
                } else if (currentPasswordTarget === 'backend') {
                    isBackendUnlocked = true;
                    switchTab('backend');
                }
                closePasswordOverlay();
            } else {
                document.getElementById('passwordError').style.display = 'block';
                document.getElementById('passwordInput').value = '';
                document.getElementById('passwordInput').focus();
            }
        }

        // 標籤切換
        function switchTab(tab) {
            // 檢查權限
            if (tab === 'observation' && !isObservationUnlocked) {
                showPasswordPrompt('observation');
                return;
            }
            if (tab === 'backend' && !isBackendUnlocked) {
                showPasswordPrompt('backend');
                return;
            }

            // 隱藏所有內容
            document.querySelectorAll('.tab-content').forEach(content => {
                content.classList.remove('active');
            });
            
            // 重置標籤樣式
            document.querySelectorAll('button[id$="-tab"]').forEach(btn => {
                btn.classList.remove('bg-blue-600', 'text-white');
                btn.classList.add('text-gray-600', 'hover:bg-gray-100');
            });
            
            // 顯示選中的內容
            document.getElementById(tab + '-content').classList.add('active');
            document.getElementById(tab + '-tab').classList.add('bg-blue-600', 'text-white');
            document.getElementById(tab + '-tab').classList.remove('text-gray-600', 'hover:bg-gray-100');
        }

        // 生成周次日期選項
        function generateWeekDateOptions() {
            const weekDateSelects = [
                document.getElementById('weekDate'),
                document.getElementById('attendanceWeekDate')
            ];
            
            const startDate = new Date(2025, 8, 1);
            const endDate = new Date(2026, 0, 20);
            const weekdays = ['日', '一', '二', '三', '四', '五', '六'];
            
            weekDateSelects.forEach(select => {
                if (!select) return;
                
                let currentDate = new Date(startDate);
                let weekNumber = 1;
                
                while (currentDate <= endDate) {
                    const weekStart = new Date(currentDate);
                    const dayOfWeek = weekStart.getDay();
                    const daysToMonday = dayOfWeek === 0 ? 6 : dayOfWeek - 1;
                    weekStart.setDate(weekStart.getDate() - daysToMonday);
                    
                    for (let i = 0; i < 7; i++) {
                        const currentDay = new Date(weekStart);
                        currentDay.setDate(weekStart.getDate() + i);
                        
                        if (currentDay > endDate) break;
                        
                        const year = currentDay.getFullYear();
                        const month = String(currentDay.getMonth() + 1).padStart(2, '0');
                        const day = String(currentDay.getDate()).padStart(2, '0');
                        const weekday = weekdays[currentDay.getDay()];
                        
                        const dateValue = `${year}-${month}-${day}`;
                        const dateText = `第${weekNumber}周 ${year}年${month}月${day}日 星期${weekday}`;
                        
                        const option = document.createElement('option');
                        option.value = dateValue;
                        option.textContent = dateText;
                        option.setAttribute('data-week', weekNumber);
                        select.appendChild(option);
                    }
                    
                    currentDate.setDate(currentDate.getDate() + 7);
                    weekNumber++;
                    
                    if (weekNumber > 21) break;
                }
            });
        }

        // 更新評量選項
        function updateEvaluationOptions() {
            updateOptionsList('participation', participationOptions, 'participationOptions', 'participationList');
            updateOptionsList('attitude', attitudeOptions, 'attitudeOptions', 'attitudeList');
            updateOptionsList('homework', homeworkOptions, 'homeworkOptions', 'homeworkList');
        }

        // 更新選項列表
        function updateOptionsList(type, options, frontendId, backendId) {
            // 更新前台檢核表
            const frontendContainer = document.getElementById(frontendId);
            if (frontendContainer) {
                frontendContainer.innerHTML = options.map(option => `
                    <label class="flex items-center space-x-2">
                        <input type="checkbox" name="${type}" value="${option}" 
                               class="rounded border-gray-300 text-blue-600 focus:ring-blue-500">
                        <span class="text-sm text-gray-700">${option}</span>
                    </label>
                `).join('');
            }
            
            // 更新後台管理列表
            const backendContainer = document.getElementById(backendId);
            if (backendContainer) {
                backendContainer.innerHTML = options.map((option, index) => `
                    <div class="flex items-center justify-between bg-gray-50 px-3 py-2 rounded-lg">
                        <span class="text-sm text-gray-700">${option}</span>
                        <button onclick="removeOption('${type}', ${index})" 
                                class="text-red-500 hover:text-red-700 text-sm transition-colors duration-200">
                            🗑️
                        </button>
                    </div>
                `).join('');
            }
        }

        // 新增評量選項
        function addOption(type) {
            const input = document.getElementById(`new${type.charAt(0).toUpperCase() + type.slice(1)}`);
            const value = input.value.trim();
            
            if (value === '') {
                alert('請輸入選項內容！');
                return;
            }
            
            let targetArray;
            let optionName = '';
            
            switch(type) {
                case 'participation':
                    targetArray = participationOptions;
                    optionName = '課堂參與度';
                    break;
                case 'attitude':
                    targetArray = attitudeOptions;
                    optionName = '學習態度';
                    break;
                case 'homework':
                    targetArray = homeworkOptions;
                    optionName = '作業完成情況';
                    break;
                default:
                    alert('未知的選項類型！');
                    return;
            }
            
            if (targetArray.includes(value)) {
                alert(`${optionName}選項「${value}」已存在！`);
                input.focus();
                return;
            }
            
            targetArray.push(value);
            input.value = '';
            updateEvaluationOptions();
            alert(`已成功新增${optionName}選項：${value}`);
        }

        // 刪除評量選項
        function removeOption(type, index) {
            let targetArray;
            let optionName = '';
            
            switch(type) {
                case 'participation':
                    targetArray = participationOptions;
                    optionName = '課堂參與度';
                    break;
                case 'attitude':
                    targetArray = attitudeOptions;
                    optionName = '學習態度';
                    break;
                case 'homework':
                    targetArray = homeworkOptions;
                    optionName = '作業完成情況';
                    break;
                default:
                    alert('未知的選項類型！');
                    return;
            }
            
            // 檢查陣列和索引的有效性
            if (!targetArray) {
                alert('選項陣列不存在！');
                return;
            }
            
            if (typeof index !== 'number' || index < 0 || index >= targetArray.length) {
                alert(`選項索引錯誤！索引: ${index}, 陣列長度: ${targetArray.length}`);
                return;
            }
            
            const optionText = targetArray[index];
            
            if (confirm(`確定要刪除${optionName}選項「${optionText}」嗎？`)) {
                try {
                    targetArray.splice(index, 1);
                    updateEvaluationOptions();
                    alert(`已成功刪除${optionName}選項：${optionText}`);
                } catch (error) {
                    alert(`刪除選項時發生錯誤：${error.message}`);
                    console.error('刪除選項錯誤:', error);
                }
            }
        }

        // 更新行為表現選項
        function updateBehaviorOptions() {
            // 更新前台下拉選單
            const select = document.getElementById('behaviorSelect');
            if (select) {
                select.innerHTML = '<option value="">請選擇行為表現</option>';
                
                behaviorOptions.forEach(option => {
                    const optionElement = document.createElement('option');
                    optionElement.value = option;
                    optionElement.textContent = option;
                    select.appendChild(optionElement);
                });
            }
            
            // 更新後台管理列表
            const container = document.getElementById('behaviorList');
            if (container) {
                container.innerHTML = behaviorOptions.map((option, index) => `
                    <div class="bg-gray-50 p-4 rounded-lg border">
                        <div class="flex items-center justify-between">
                            <span class="text-gray-800">${option}</span>
                            <button onclick="removeBehavior(${index})" 
                                    class="text-red-500 hover:text-red-700 transition-colors duration-200">
                                🗑️
                            </button>
                        </div>
                    </div>
                `).join('');
            }
        }

        // 新增行為表現選項
        function addBehavior() {
            const input = document.getElementById('newBehavior');
            const value = input.value.trim();
            
            if (value === '') {
                alert('請輸入行為表現描述！');
                return;
            }
            
            if (behaviorOptions.includes(value)) {
                alert('選項已存在！');
                return;
            }
            
            behaviorOptions.push(value);
            input.value = '';
            updateBehaviorOptions();
            alert(`已新增行為表現選項：${value}`);
        }

        // 刪除行為表現選項
        function removeBehavior(index) {
            // 檢查索引的有效性
            if (typeof index !== 'number' || index < 0 || index >= behaviorOptions.length) {
                alert(`選項索引錯誤！索引: ${index}, 陣列長度: ${behaviorOptions.length}`);
                return;
            }
            
            const option = behaviorOptions[index];
            
            if (confirm(`確定要刪除行為表現選項「${option}」嗎？`)) {
                try {
                    behaviorOptions.splice(index, 1);
                    updateBehaviorOptions();
                    alert(`已成功刪除行為表現選項：${option}`);
                } catch (error) {
                    alert(`刪除選項時發生錯誤：${error.message}`);
                    console.error('刪除行為選項錯誤:', error);
                }
            }
        }

        // 提交點名記錄
        function submitAttendance() {
            const teacher = document.getElementById('attendanceTeacher').value;
            const grade = document.getElementById('attendanceGrade').value;
            const weekDate = document.getElementById('attendanceWeekDate').value;
            const period = document.getElementById('attendancePeriod').value;
            const course = document.getElementById('attendanceCourse').value;
            const studentName = document.getElementById('attendanceStudentName').value.trim();
            
            if (!teacher || !grade || !weekDate || !period || !course || !studentName) {
                alert('請完整填寫所有欄位！');
                return;
            }
            
            // 獲取出席狀態
            const attendanceStatus = document.querySelector('input[name="attendanceStatus"]:checked').value;
            const leaveReason = document.getElementById('leaveReason').value.trim();
            const lateReason = document.getElementById('lateReason').value.trim();
            
            // 如果選擇請假但沒填原因，提醒填寫
            if (attendanceStatus === '請假' && !leaveReason) {
                alert('請填寫請假原因！');
                document.getElementById('leaveReason').focus();
                return;
            }
            
            // 如果選擇遲到但沒填原因，提醒填寫
            if (attendanceStatus === '遲到' && !lateReason) {
                alert('請填寫遲到原因！');
                document.getElementById('lateReason').focus();
                return;
            }
            
            // 檢查是否重複
            const isDuplicate = attendanceRecords.some(record => 
                record.weekDate === weekDate && 
                record.period === period && 
                record.course === course && 
                record.grade === grade &&
                record.studentName === studentName
            );
            
            if (isDuplicate) {
                alert('此學生在相同課程時段已有點名記錄！');
                return;
            }
            
            // 記錄送出時間
            const submitTime = new Date().toLocaleString('zh-TW', {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit'
            });
            
            // 獲取周次資訊
            const selectedOption = document.getElementById('attendanceWeekDate').selectedOptions[0];
            const weekNumber = selectedOption.getAttribute('data-week');
            
            // 建立記錄
            const record = {
                teacher: teacher,
                grade: grade,
                weekDate: weekDate,
                weekDateText: selectedOption.text,
                weekNumber: weekNumber,
                period: period,
                course: course,
                studentName: studentName,
                attendanceStatus: attendanceStatus,
                leaveReason: attendanceStatus === '請假' ? leaveReason : '',
                lateReason: attendanceStatus === '遲到' ? lateReason : '',
                submitTime: submitTime,
                type: 'attendance'
            };
            
            attendanceRecords.push(record);
            
            // 清空學生姓名欄位和重置狀態
            document.getElementById('attendanceStudentName').value = '';
            document.querySelector('input[name="attendanceStatus"][value="出席"]').checked = true;
            document.getElementById('leaveReason').value = '';
            document.getElementById('leaveReason').disabled = true;
            document.getElementById('lateReason').value = '';
            document.getElementById('lateReason').disabled = true;
            
            updateAttendanceRecordsList();
            updateAllRecordsList();
            
            // 同步觀察記錄表單
            syncObservationFormWithAttendance();
            
            alert(`${studentName} 的點名記錄已成功提交！`);
        }

        // 提交觀察記錄
        function submitObservation() {
            const teacher = document.getElementById('observationTeacher').value;
            const grade = document.getElementById('observationGrade').value;
            const weekDate = document.getElementById('weekDate').value;
            const period = document.getElementById('period').value;
            const course = document.getElementById('course').value;
            const studentName = document.getElementById('studentName').value.trim();
            
            if (!teacher || !grade || !weekDate || !period || !course || !studentName) {
                alert('請完整填寫基本資訊！');
                return;
            }
            
            // 檢查是否重複
            const isDuplicate = observationRecords.some(record => 
                record.weekDate === weekDate && 
                record.period === period && 
                record.course === course && 
                record.grade === grade &&
                record.studentName === studentName
            );
            
            if (isDuplicate) {
                alert('此學生在相同課程時段已有觀察記錄！');
                return;
            }
            
            // 收集觀察記錄
            const participation = Array.from(document.querySelectorAll('input[name="participation"]:checked')).map(cb => cb.value);
            const attitude = Array.from(document.querySelectorAll('input[name="attitude"]:checked')).map(cb => cb.value);
            const homework = Array.from(document.querySelectorAll('input[name="homework"]:checked')).map(cb => cb.value);
            const behavior = document.getElementById('behaviorSelect').value;
            const specialEvent = document.getElementById('specialEvent').value.trim();
            
            // 記錄送出時間
            const submitTime = new Date().toLocaleString('zh-TW', {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit'
            });
            
            // 獲取周次資訊
            const selectedOption = document.getElementById('weekDate').selectedOptions[0];
            const weekNumber = selectedOption.getAttribute('data-week');
            
            // 建立記錄
            const record = {
                teacher: teacher,
                grade: grade,
                weekDate: weekDate,
                weekDateText: selectedOption.text,
                weekNumber: weekNumber,
                period: period,
                course: course,
                studentName: studentName,
                participation: participation,
                attitude: attitude,
                homework: homework,
                behavior: behavior,
                specialEvent: specialEvent,
                submitTime: submitTime,
                type: 'observation'
            };
            
            observationRecords.push(record);
            
            // 清空表單（保留基本資訊，只清空學生相關欄位）
            document.getElementById('studentName').value = '';
            document.querySelectorAll('input[type="checkbox"]').forEach(cb => cb.checked = false);
            document.getElementById('behaviorSelect').value = '';
            document.getElementById('specialEvent').value = '';
            
            updateObservationRecordsList();
            updateAllRecordsList();
            
            alert(`${studentName} 的觀察記錄已成功提交！`);
        }

        // 更新點名記錄列表
        function updateAttendanceRecordsList() {
            const container = document.getElementById('attendanceRecordsList');
            if (!container) return;
            
            if (attendanceRecords.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-4">尚無點名記錄</p>';
                return;
            }
            
            // 取得第一筆記錄的課程資訊
            const firstRecord = attendanceRecords[0];
            
            let html = `
                <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400 mb-4">
                    <div class="text-sm font-semibold text-gray-700 mb-2">課程資訊</div>
                    <div class="text-sm text-gray-600">
                        授課教師: ${firstRecord.teacher}<br>
                        年級: ${firstRecord.grade}<br>
                        ${firstRecord.weekDateText}<br>
                        上課節次: ${firstRecord.period}<br>
                        課程組別: ${firstRecord.course}
                    </div>
                </div>
            `;
            
            // 顯示所有學生點名記錄
            html += attendanceRecords.map((record, index) => {
                let statusIcon = '✅';
                if (record.attendanceStatus === '遲到') statusIcon = '⏰';
                else if (record.attendanceStatus === '請假') statusIcon = '🏠';
                
                let statusText = record.attendanceStatus;
                if (record.attendanceStatus === '請假' && record.leaveReason) {
                    statusText = `${record.attendanceStatus}（${record.leaveReason}）`;
                } else if (record.attendanceStatus === '遲到' && record.lateReason) {
                    statusText = `${record.attendanceStatus}（${record.lateReason}）`;
                }
                
                return `
                    <div class="bg-green-50 p-4 rounded-lg border-l-4 border-green-400">
                        <div class="font-medium text-gray-800 mb-2">${index + 1}. ${record.studentName} ${statusIcon} ${statusText}</div>
                        <div class="text-xs text-gray-500">點名時間: ${record.submitTime}</div>
                    </div>
                `;
            }).join('');
            
            container.innerHTML = html;
        }

        // 更新觀察記錄列表
        function updateObservationRecordsList() {
            const container = document.getElementById('observationRecordsList');
            if (!container) return;
            
            if (observationRecords.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-4">尚無觀察記錄</p>';
                return;
            }
            
            // 取得第一筆記錄的課程資訊
            const firstRecord = observationRecords[0];
            
            let html = `
                <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400 mb-4">
                    <div class="text-sm font-semibold text-gray-700 mb-2">課程資訊</div>
                    <div class="text-sm text-gray-600">
                        授課教師: ${firstRecord.teacher}<br>
                        年級: ${firstRecord.grade}<br>
                        ${firstRecord.weekDateText}<br>
                        上課節次: ${firstRecord.period}<br>
                        課程組別: ${firstRecord.course}
                    </div>
                </div>
            `;
            
            // 顯示所有學生觀察記錄
            html += observationRecords.map((record, index) => {
                const participationText = record.participation.length > 0 ? record.participation.join(', ') : '未填寫';
                const attitudeText = record.attitude.length > 0 ? record.attitude.join(', ') : '未填寫';
                const homeworkText = record.homework.length > 0 ? record.homework.join(', ') : '未填寫';
                
                return `
                    <div class="bg-green-50 p-4 rounded-lg border-l-4 border-green-400">
                        <div class="font-medium text-gray-800 mb-2">${index + 1}. ${record.studentName} ✅</div>
                        <div class="text-sm text-gray-600 space-y-1">
                            <div><strong>參與度：</strong>${participationText}</div>
                            <div><strong>學習態度：</strong>${attitudeText}</div>
                            <div><strong>作業完成：</strong>${homeworkText}</div>
                            <div><strong>行為表現：</strong>${record.behavior || '未填寫'}</div>
                            ${record.specialEvent ? `<div><strong>特殊事件：</strong>${record.specialEvent}</div>` : ''}
                            <div class="text-xs text-gray-500">送出時間: ${record.submitTime}</div>
                        </div>
                    </div>
                `;
            }).join('');
            
            container.innerHTML = html;
        }

        // 彙整同一堂課的點名記錄
        function consolidateAttendanceRecords() {
            const courseGroups = {};
            
            // 按課程組別、日期、節次分組
            attendanceRecords.forEach(record => {
                if (record.isIntegratedRecord) return; // 跳過已經是整合記錄的
                
                const courseKey = `${record.course}_${record.weekDate}_${record.period}`;
                
                if (!courseGroups[courseKey]) {
                    courseGroups[courseKey] = {
                        courseKey: courseKey,
                        teacher: record.teacher,
                        grade: record.grade,
                        weekDate: record.weekDate,
                        weekDateText: record.weekDateText,
                        period: record.period,
                        course: record.course,
                        students: [],
                        type: 'attendance',
                        isConsolidated: true
                    };
                }
                
                courseGroups[courseKey].students.push({
                    studentName: record.studentName,
                    attendanceStatus: record.attendanceStatus,
                    leaveReason: record.leaveReason,
                    submitTime: record.submitTime
                });
            });
            
            // 轉換為陣列並計算統計資料
            return Object.values(courseGroups).map(group => {
                const totalStudents = group.students.length;
                const presentCount = group.students.filter(s => s.attendanceStatus === '出席').length;
                const lateCount = group.students.filter(s => s.attendanceStatus === '遲到').length;
                const absentCount = group.students.filter(s => s.attendanceStatus === '請假').length;
                const lastUpdateTime = group.students.reduce((latest, student) => {
                    return new Date(student.submitTime) > new Date(latest) ? student.submitTime : latest;
                }, group.students[0]?.submitTime || '');
                
                return {
                    ...group,
                    totalStudents,
                    presentCount,
                    lateCount,
                    absentCount,
                    lastUpdateTime
                };
            });
        }

        // 更新所有記錄列表（後台）
        function updateAllRecordsList() {
            const container = document.getElementById('allRecordsList');
            if (!container) return;
            
            // 彙整同一堂課的點名記錄
            const consolidatedAttendanceRecords = consolidateAttendanceRecords();
            const allRecords = [...consolidatedAttendanceRecords, ...observationRecords];
            
            if (allRecords.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-4">尚無記錄</p>';
                return;
            }
            
            container.innerHTML = allRecords.map((record, index) => {
                if (record.type === 'attendance') {
                    if (record.isConsolidated) {
                        // 彙整記錄顯示
                        return `
                            <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400">
                                <div class="flex items-start justify-between">
                                    <div class="flex-1">
                                        <div class="font-medium text-gray-800 mb-2">
                                            【課程點名彙整】${record.course} - ${record.teacher} - ${record.grade}
                                        </div>
                                        <div class="text-sm text-gray-600 mb-2">
                                            ${record.weekDateText} 上課節次: ${record.period}
                                        </div>
                                        <div class="text-sm text-gray-600 mb-2">
                                            共 ${record.totalStudents} 位學生 | 
                                            ✅ 出席: ${record.presentCount} 人 | 
                                            ⏰ 遲到: ${record.lateCount} 人 | 
                                            🏠 請假: ${record.absentCount} 人
                                        </div>
                                        <div class="text-xs text-gray-500">最後更新: ${record.lastUpdateTime}</div>
                                    </div>
                                    <button onclick="removeConsolidatedRecord('${record.courseKey}')" 
                                            class="text-red-500 hover:text-red-700 transition-colors duration-200 ml-4">
                                        🗑️
                                    </button>
                                </div>
                            </div>
                        `;
                    } else {
                        // 個別記錄顯示（如果有未彙整的）
                        let statusIcon = '✅';
                        if (record.attendanceStatus === '遲到') statusIcon = '⏰';
                        else if (record.attendanceStatus === '請假') statusIcon = '🏠';
                        
                        let statusText = record.attendanceStatus;
                        if (record.attendanceStatus === '請假' && record.leaveReason) {
                            statusText = `${record.attendanceStatus}（${record.leaveReason}）`;
                        } else if (record.attendanceStatus === '遲到' && record.lateReason) {
                            statusText = `${record.attendanceStatus}（${record.lateReason}）`;
                        }
                        
                        return `
                            <div class="bg-gray-50 p-4 rounded-lg border">
                                <div class="flex items-start justify-between">
                                    <div class="flex-1">
                                        <div class="font-medium text-gray-800 mb-2">
                                            【點名】${record.studentName} ${statusIcon} ${statusText} - ${record.teacher} - ${record.grade} ${record.course}
                                        </div>
                                        <div class="text-sm text-gray-600 mb-1">
                                            ${record.weekDateText} 上課節次: ${record.period}
                                        </div>
                                        <div class="text-xs text-gray-500">點名時間: ${record.submitTime}</div>
                                    </div>
                                    <button onclick="removeAllRecord(${index})" 
                                            class="text-red-500 hover:text-red-700 transition-colors duration-200 ml-4">
                                        🗑️
                                    </button>
                                </div>
                            </div>
                        `;
                    }
                } else {
                    const participationText = record.participation.length > 0 ? record.participation.join(', ') : '未填寫';
                    const attitudeText = record.attitude.length > 0 ? record.attitude.join(', ') : '未填寫';
                    const homeworkText = record.homework.length > 0 ? record.homework.join(', ') : '未填寫';
                    
                    return `
                        <div class="bg-gray-50 p-4 rounded-lg border">
                            <div class="flex items-start justify-between">
                                <div class="flex-1">
                                    <div class="font-medium text-gray-800 mb-2">
                                        【觀察】${record.studentName} - ${record.teacher} - ${record.grade} ${record.course}
                                    </div>
                                    <div class="text-sm text-gray-600 mb-2">
                                        ${record.weekDateText} 上課節次: ${record.period}
                                    </div>
                                    <div class="text-sm text-gray-600 space-y-1">
                                        <div><strong>參與度：</strong>${participationText}</div>
                                        <div><strong>學習態度：</strong>${attitudeText}</div>
                                        <div><strong>作業完成：</strong>${homeworkText}</div>
                                        <div><strong>行為表現：</strong>${record.behavior || '未填寫'}</div>
                                        ${record.specialEvent ? `<div><strong>特殊事件：</strong>${record.specialEvent}</div>` : ''}
                                        <div class="text-xs text-gray-500">送出時間: ${record.submitTime}</div>
                                    </div>
                                </div>
                                <button onclick="removeAllRecord(${index})" 
                                        class="text-red-500 hover:text-red-700 transition-colors duration-200 ml-4">
                                    🗑️
                                </button>
                            </div>
                        </div>
                    `;
                }
            }).join('');
        }

        // 刪除彙整記錄
        function removeConsolidatedRecord(courseKey) {
            if (confirm('確定要刪除此課程的所有點名記錄嗎？')) {
                // 刪除該課程的所有點名記錄
                attendanceRecords = attendanceRecords.filter(record => {
                    const recordCourseKey = `${record.course}_${record.weekDate}_${record.period}`;
                    return recordCourseKey !== courseKey;
                });
                
                updateAttendanceRecordsList();
                updateObservationRecordsList();
                updateAllRecordsList();
                alert('課程點名記錄已刪除！');
            }
        }

        // 刪除記錄（僅後台可用）
        function removeAllRecord(index) {
            const consolidatedAttendanceRecords = consolidateAttendanceRecords();
            const allRecords = [...consolidatedAttendanceRecords, ...observationRecords];
            const record = allRecords[index];
            
            if (record.type === 'attendance' && !record.isConsolidated) {
                if (confirm(`確定要刪除「${record.studentName}」的記錄嗎？`)) {
                    const attendanceIndex = attendanceRecords.findIndex(r => r === record);
                    attendanceRecords.splice(attendanceIndex, 1);
                    
                    updateAttendanceRecordsList();
                    updateObservationRecordsList();
                    updateAllRecordsList();
                }
            } else if (record.type === 'observation') {
                if (confirm(`確定要刪除「${record.studentName}」的觀察記錄嗎？`)) {
                    const observationIndex = observationRecords.findIndex(r => r === record);
                    observationRecords.splice(observationIndex, 1);
                    
                    updateAttendanceRecordsList();
                    updateObservationRecordsList();
                    updateAllRecordsList();
                }
            }
        }

        // 清空所有記錄
        function clearAllRecords() {
            if (confirm('確定要清空所有記錄嗎？此操作無法復原！')) {
                attendanceRecords = [];
                observationRecords = [];
                updateAttendanceRecordsList();
                updateObservationRecordsList();
                updateAllRecordsList();
                alert('所有記錄已清空！');
            }
        }

        // 儲存點名記錄到記錄管理
        function saveAttendanceRecords() {
            if (attendanceRecords.length === 0) {
                alert('沒有可儲存的點名記錄！');
                return;
            }
            
            // 初始化儲存記錄陣列
            if (!window.savedRecords) window.savedRecords = [];
            
            // 以課程組別為主建立記錄
            const firstRecord = attendanceRecords[0];
            const courseKey = `${firstRecord.course}_${firstRecord.weekDate}_${firstRecord.period}`;
            
            // 查找是否已有該課程的記錄
            let existingRecord = window.savedRecords.find(r => r.courseKey === courseKey);
            
            if (!existingRecord) {
                // 建立新的課程記錄
                existingRecord = {
                    id: Date.now() + Math.random(),
                    courseKey: courseKey,
                    title: `${firstRecord.course} - ${firstRecord.weekDateText} ${firstRecord.period}`,
                    teacher: firstRecord.teacher,
                    grade: firstRecord.grade,
                    weekDate: firstRecord.weekDate,
                    weekDateText: firstRecord.weekDateText,
                    period: firstRecord.period,
                    course: firstRecord.course,
                    students: [],
                    createTime: new Date().toLocaleString('zh-TW'),
                    updateTime: new Date().toLocaleString('zh-TW'),
                    type: 'attendance'
                };
                window.savedRecords.push(existingRecord);
            }
            
            // 統計學生出席狀況
            attendanceRecords.forEach(record => {
                const studentData = {
                    studentName: record.studentName,
                    attendanceStatus: record.attendanceStatus,
                    leaveReason: record.leaveReason,
                    lateReason: record.lateReason,
                    submitTime: record.submitTime
                };
                
                // 檢查是否已存在該學生記錄
                const existingStudent = existingRecord.students.find(s => s.studentName === record.studentName);
                if (!existingStudent) {
                    existingRecord.students.push(studentData);
                }
            });
            
            existingRecord.updateTime = new Date().toLocaleString('zh-TW');
            
            // 同時將整合記錄加入原始記錄（即時資料）- 同一堂課只儲存一筆
            const existingRawRecord = attendanceRecords.find(r => 
                r.course === firstRecord.course && 
                r.weekDate === firstRecord.weekDate && 
                r.period === firstRecord.period &&
                r.isIntegratedRecord === true
            );
            
            if (!existingRawRecord) {
                // 建立整合記錄加入原始記錄
                const integratedRecord = {
                    teacher: firstRecord.teacher,
                    grade: firstRecord.grade,
                    weekDate: firstRecord.weekDate,
                    weekDateText: firstRecord.weekDateText,
                    weekNumber: firstRecord.weekNumber,
                    period: firstRecord.period,
                    course: firstRecord.course,
                    studentName: `【課程整合記錄】共${existingRecord.students.length}位學生`,
                    attendanceStatus: `出席:${existingRecord.students.filter(s => s.attendanceStatus === '出席').length} 遲到:${existingRecord.students.filter(s => s.attendanceStatus === '遲到').length} 請假:${existingRecord.students.filter(s => s.attendanceStatus === '請假').length}`,
                    leaveReason: '',
                    submitTime: new Date().toLocaleString('zh-TW'),
                    type: 'attendance',
                    isIntegratedRecord: true,
                    studentsDetail: existingRecord.students
                };
                
                attendanceRecords.push(integratedRecord);
            }
            
            updateRecordManagementList();
            updateAllRecordsList();
            
            // 顯示成功訊息並自動消失
            showSuccessMessage('儲存記錄成功！已同步至權限控制與設定的原始記錄');
        }

        // 儲存觀察記錄到記錄管理
        function saveObservationRecords() {
            if (observationRecords.length === 0) {
                alert('沒有可儲存的觀察記錄！');
                return;
            }
            
            // 初始化儲存記錄陣列
            if (!window.savedRecords) window.savedRecords = [];
            
            // 以課程組別為主建立記錄
            const firstRecord = observationRecords[0];
            const courseKey = `${firstRecord.course}_${firstRecord.weekDate}_${firstRecord.period}_observation`;
            
            // 查找是否已有該課程的觀察記錄
            let existingRecord = window.savedRecords.find(r => r.courseKey === courseKey);
            
            if (!existingRecord) {
                // 建立新的課程觀察記錄
                existingRecord = {
                    id: Date.now() + Math.random(),
                    courseKey: courseKey,
                    title: `${firstRecord.course} - ${firstRecord.weekDateText} ${firstRecord.period} (觀察記錄)`,
                    teacher: firstRecord.teacher,
                    grade: firstRecord.grade,
                    weekDate: firstRecord.weekDate,
                    weekDateText: firstRecord.weekDateText,
                    period: firstRecord.period,
                    course: firstRecord.course,
                    students: [],
                    createTime: new Date().toLocaleString('zh-TW'),
                    updateTime: new Date().toLocaleString('zh-TW'),
                    type: 'observation'
                };
                window.savedRecords.push(existingRecord);
            }
            
            // 統計學生觀察記錄
            observationRecords.forEach(record => {
                const studentData = {
                    studentName: record.studentName,
                    participation: record.participation,
                    attitude: record.attitude,
                    homework: record.homework,
                    behavior: record.behavior,
                    specialEvent: record.specialEvent,
                    submitTime: record.submitTime
                };
                
                // 檢查是否已存在該學生記錄
                const existingStudent = existingRecord.students.find(s => s.studentName === record.studentName);
                if (!existingStudent) {
                    existingRecord.students.push(studentData);
                }
            });
            
            existingRecord.updateTime = new Date().toLocaleString('zh-TW');
            
            updateRecordManagementList();
            updateAllRecordsList();
            
            // 顯示成功訊息並自動消失
            showSuccessMessage('儲存觀察記錄成功！已同步至權限控制與設定的原始記錄');
        }

        // 密碼變更功能
        function changePassword() {
            const currentPassword = document.getElementById('currentPassword').value;
            const newPassword = document.getElementById('newPassword').value;
            const confirmPassword = document.getElementById('confirmPassword').value;
            
            // 驗證輸入
            if (!currentPassword || !newPassword || !confirmPassword) {
                alert('請填寫所有密碼欄位！');
                return;
            }
            
            // 驗證目前密碼
            if (currentPassword !== TEACHER_PASSWORD) {
                alert('目前密碼錯誤！');
                document.getElementById('currentPassword').focus();
                return;
            }
            
            // 驗證新密碼長度
            if (newPassword.length < 6) {
                alert('新密碼長度至少需要6個字元！');
                document.getElementById('newPassword').focus();
                return;
            }
            
            // 驗證密碼確認
            if (newPassword !== confirmPassword) {
                alert('新密碼與確認密碼不一致！');
                document.getElementById('confirmPassword').focus();
                return;
            }
            
            // 更新密碼
            TEACHER_PASSWORD = newPassword;
            
            // 清空表單
            document.getElementById('currentPassword').value = '';
            document.getElementById('newPassword').value = '';
            document.getElementById('confirmPassword').value = '';
            
            alert('密碼已成功更新！請記住您的新密碼。');
        }

        // 上傳學校標誌功能
        function uploadSchoolLogo() {
            const fileInput = document.getElementById('schoolLogoFile');
            const file = fileInput.files[0];
            
            if (!file) {
                alert('請選擇要上傳的學校標誌圖片！');
                return;
            }
            
            // 檢查檔案類型
            if (!file.type.startsWith('image/')) {
                alert('請選擇有效的圖片檔案！');
                return;
            }
            
            // 檢查檔案大小 (限制2MB)
            if (file.size > 2 * 1024 * 1024) {
                alert('圖片檔案過大，請選擇小於2MB的圖片！');
                return;
            }
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const imageDataUrl = e.target.result;
                setSchoolLogo(imageDataUrl);
                alert('學校標誌已成功上傳並設定！');
            };
            
            reader.onerror = function() {
                alert('圖片讀取失敗，請重新選擇檔案！');
            };
            
            reader.readAsDataURL(file);
        }

        // 設定學校標誌
        function setSchoolLogo(imageSource) {
            const logoContainer = document.getElementById('schoolLogoContainer');
            const logoImage = document.getElementById('schoolLogoImage');
            const previewSection = document.getElementById('logoPreviewSection');
            const previewImage = document.getElementById('logoPreviewImage');
            
            // 設定主標題區域的標誌
            logoImage.src = imageSource;
            logoContainer.style.display = 'block';
            
            // 設定預覽區域
            previewImage.src = imageSource;
            previewSection.style.display = 'block';
        }

        // 移除學校標誌
        function removeSchoolLogo() {
            const logoContainer = document.getElementById('schoolLogoContainer');
            const logoImage = document.getElementById('schoolLogoImage');
            const previewSection = document.getElementById('logoPreviewSection');
            const previewImage = document.getElementById('logoPreviewImage');
            const fileInput = document.getElementById('schoolLogoFile');
            
            // 隱藏標誌
            logoContainer.style.display = 'none';
            logoImage.src = '';
            
            // 隱藏預覽
            previewSection.style.display = 'none';
            previewImage.src = '';
            
            // 清空檔案選擇器
            if (fileInput) fileInput.value = '';
            
            alert('學校標誌已移除！');
        }

        // 上傳本地圖片功能
        function uploadBackgroundImage() {
            const fileInput = document.getElementById('backgroundImageFile');
            const file = fileInput.files[0];
            
            if (!file) {
                alert('請選擇要上傳的圖片檔案！');
                return;
            }
            
            // 檢查檔案類型
            if (!file.type.startsWith('image/')) {
                alert('請選擇有效的圖片檔案！');
                return;
            }
            
            // 檢查檔案大小 (限制10MB)
            if (file.size > 10 * 1024 * 1024) {
                alert('圖片檔案過大，請選擇小於10MB的圖片！');
                return;
            }
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const imageDataUrl = e.target.result;
                setBackgroundImageFromData(imageDataUrl);
                alert('背景圖片已成功上傳並設定！');
            };
            
            reader.onerror = function() {
                alert('圖片讀取失敗，請重新選擇檔案！');
            };
            
            reader.readAsDataURL(file);
        }

        // 從網址設定背景圖
        function setBackgroundImageFromUrl() {
            const imageUrl = document.getElementById('backgroundImageUrl').value.trim();
            
            if (!imageUrl) {
                alert('請輸入圖片網址！');
                return;
            }
            
            // 測試圖片是否有效
            const img = new Image();
            img.onload = function() {
                setBackgroundImageFromData(imageUrl);
                alert('背景圖已成功設定！');
            };
            
            img.onerror = function() {
                alert('圖片載入失敗，請檢查網址是否正確！');
            };
            
            img.src = imageUrl;
        }

        // 設定背景圖片的通用函數
        function setBackgroundImageFromData(imageSource) {
            const opacity = document.getElementById('backgroundOpacity').value;
            
            // 設定背景圖片
            document.body.style.backgroundImage = `url('${imageSource}')`;
            document.body.style.backgroundSize = 'cover';
            document.body.style.backgroundPosition = 'center';
            document.body.style.backgroundRepeat = 'no-repeat';
            document.body.style.backgroundAttachment = 'fixed';
            
            // 添加半透明覆蓋層
            updateBackgroundOverlay(opacity);
        }

        // 更新背景覆蓋層
        function updateBackgroundOverlay(opacity) {
            let overlay = document.getElementById('backgroundOverlay');
            
            if (!overlay) {
                overlay = document.createElement('div');
                overlay.id = 'backgroundOverlay';
                overlay.style.position = 'fixed';
                overlay.style.top = '0';
                overlay.style.left = '0';
                overlay.style.width = '100%';
                overlay.style.height = '100%';
                overlay.style.zIndex = '-1';
                overlay.style.pointerEvents = 'none';
                document.body.appendChild(overlay);
            }
            
            overlay.style.backgroundColor = `rgba(255, 255, 255, ${1 - opacity})`;
        }

        // 套用透明度
        function applyOpacity() {
            const opacity = document.getElementById('backgroundOpacity').value;
            
            // 檢查是否有背景圖
            if (!document.body.style.backgroundImage) {
                alert('請先設定背景圖片！');
                return;
            }
            
            updateBackgroundOverlay(opacity);
            alert('透明度已套用！');
        }

        // 更新透明度顯示
        function updateOpacityDisplay() {
            const opacity = document.getElementById('backgroundOpacity').value;
            document.getElementById('opacityValue').textContent = opacity;
        }
        
        function removeBackgroundImage() {
            document.body.style.backgroundImage = '';
            document.body.style.backgroundSize = '';
            document.body.style.backgroundPosition = '';
            document.body.style.backgroundRepeat = '';
            document.body.style.backgroundAttachment = '';
            
            const overlay = document.getElementById('backgroundOverlay');
            if (overlay) {
                overlay.remove();
            }
            
            // 清空檔案選擇器和網址輸入框
            const fileInput = document.getElementById('backgroundImageFile');
            const urlInput = document.getElementById('backgroundImageUrl');
            if (fileInput) fileInput.value = '';
            if (urlInput) urlInput.value = '';
            
            alert('背景圖已移除！');
        }

        // 更新記錄管理列表
        function updateRecordManagementList() {
            const container = document.getElementById('recordManagementList');
            if (!container) return;
            
            if (!window.savedRecords || window.savedRecords.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-4">尚無儲存的課程記錄</p>';
                return;
            }
            
            container.innerHTML = window.savedRecords.map((record, index) => {
                const totalStudents = record.students ? record.students.length : 0;
                
                if (record.type === 'observation') {
                    // 觀察記錄顯示
                    return `
                        <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-4 rounded-lg border-l-4 border-purple-400">
                            <div class="flex items-start justify-between">
                                <div class="flex-1">
                                    <div class="font-medium text-gray-800 mb-2">📊 ${record.title}</div>
                                    <div class="text-sm text-gray-600 mb-2">
                                        👨‍🏫 授課教師：${record.teacher} | 🎓 年級：${record.grade}
                                    </div>
                                    <div class="text-sm text-gray-600 mb-2">
                                        👥 觀察學生數：${totalStudents} 人
                                    </div>
                                    <div class="text-xs text-gray-500">
                                        建立時間: ${record.createTime} | 更新時間: ${record.updateTime}
                                    </div>
                                </div>
                                <div class="flex gap-2 ml-4">
                                    <button onclick="exportSingleCourseRecord(${index})" 
                                            class="px-3 py-1 bg-green-600 text-white rounded text-xs hover:bg-green-700 transition-colors duration-200">
                                        📊 匯出
                                    </button>
                                    <button onclick="removeSavedRecord(${index})" 
                                            class="px-3 py-1 bg-red-500 text-white rounded text-xs hover:bg-red-600 transition-colors duration-200">
                                        🗑️
                                    </button>
                                </div>
                            </div>
                        </div>
                    `;
                } else {
                    // 點名記錄顯示
                    const presentCount = record.students ? record.students.filter(s => s.attendanceStatus === '出席').length : 0;
                    const lateCount = record.students ? record.students.filter(s => s.attendanceStatus === '遲到').length : 0;
                    const absentCount = record.students ? record.students.filter(s => s.attendanceStatus === '請假').length : 0;
                    
                    return `
                        <div class="bg-gradient-to-r from-blue-50 to-green-50 p-4 rounded-lg border-l-4 border-indigo-400">
                            <div class="flex items-start justify-between">
                                <div class="flex-1">
                                    <div class="font-medium text-gray-800 mb-2">📚 ${record.title}</div>
                                    <div class="text-sm text-gray-600 mb-2">
                                        👨‍🏫 授課教師：${record.teacher} | 🎓 年級：${record.grade}
                                    </div>
                                    <div class="text-sm text-gray-600 mb-2">
                                        👥 總學生數：${totalStudents} 人 | 
                                        ✅ 出席：${presentCount} 人 | 
                                        ⏰ 遲到：${lateCount} 人 | 
                                        🏠 請假：${absentCount} 人
                                    </div>
                                    <div class="text-xs text-gray-500">
                                        建立時間: ${record.createTime} | 更新時間: ${record.updateTime}
                                    </div>
                                </div>
                                <div class="flex gap-2 ml-4">
                                    <button onclick="exportSingleCourseRecord(${index})" 
                                            class="px-3 py-1 bg-green-600 text-white rounded text-xs hover:bg-green-700 transition-colors duration-200">
                                        📊 匯出
                                    </button>
                                    <button onclick="removeSavedRecord(${index})" 
                                            class="px-3 py-1 bg-red-500 text-white rounded text-xs hover:bg-red-600 transition-colors duration-200">
                                        🗑️
                                    </button>
                                </div>
                            </div>
                        </div>
                    `;
                }
            }).join('');
        }

        // 匯出單一課程記錄
        function exportSingleCourseRecord(index) {
            if (!window.savedRecords || !window.savedRecords[index]) {
                alert('記錄不存在！');
                return;
            }
            
            const record = window.savedRecords[index];
            const workbook = XLSX.utils.book_new();
            
            if (record.type === 'observation') {
                // 觀察記錄工作表
                const observationData = [
                    [`${record.title} - 課堂觀察記錄`],
                    [''],
                    ['課程資訊'],
                    ['授課教師', record.teacher],
                    ['年級', record.grade],
                    ['課程組別', record.course],
                    ['上課時間', `${record.weekDateText} ${record.period}`],
                    [''],
                    ['學生觀察記錄'],
                    ['序號', '學生姓名', '課堂參與度', '學習態度', '作業完成情況', '行為表現', '特殊事件記錄', '記錄時間']
                ];
                
                if (record.students && record.students.length > 0) {
                    record.students.forEach((student, idx) => {
                        observationData.push([
                            idx + 1,
                            student.studentName,
                            student.participation.join(', ') || '未填寫',
                            student.attitude.join(', ') || '未填寫',
                            student.homework.join(', ') || '未填寫',
                            student.behavior || '未填寫',
                            student.specialEvent || '未填寫',
                            student.submitTime
                        ]);
                    });
                    
                    observationData.push(['']);
                    observationData.push(['觀察統計']);
                    observationData.push(['觀察學生數', record.students.length]);
                }
                
                observationData.push(['']);
                observationData.push(['匯出時間', new Date().toLocaleString('zh-TW')]);
                
                const observationWorksheet = XLSX.utils.aoa_to_sheet(observationData);
                observationWorksheet['!cols'] = [
                    { width: 8 }, { width: 15 }, { width: 20 }, { width: 20 }, 
                    { width: 20 }, { width: 15 }, { width: 30 }, { width: 20 }
                ];
                XLSX.utils.book_append_sheet(workbook, observationWorksheet, '課程觀察記錄');
                
                const fileName = `課程觀察記錄_${record.course}_${record.weekDate}_${new Date().toISOString().split('T')[0]}.xlsx`;
                XLSX.writeFile(workbook, fileName);
            } else {
                // 課程出席記錄工作表
                const attendanceData = [
                    [`${record.title} - 課堂出席記錄`],
                    [''],
                    ['課程資訊'],
                    ['授課教師', record.teacher],
                    ['年級', record.grade],
                    ['課程組別', record.course],
                    ['上課時間', `${record.weekDateText} ${record.period}`],
                    [''],
                    ['學生出席狀況'],
                    ['序號', '學生姓名', '出席狀態', '請假原因', '點名時間']
                ];
                
                if (record.students && record.students.length > 0) {
                    record.students.forEach((student, idx) => {
                        attendanceData.push([
                            idx + 1,
                            student.studentName,
                            student.attendanceStatus,
                            student.leaveReason || '',
                            student.submitTime
                        ]);
                    });
                    
                    // 統計資料
                    const totalStudents = record.students.length;
                    const presentCount = record.students.filter(s => s.attendanceStatus === '出席').length;
                    const lateCount = record.students.filter(s => s.attendanceStatus === '遲到').length;
                    const absentCount = record.students.filter(s => s.attendanceStatus === '請假').length;
                    
                    attendanceData.push(['']);
                    attendanceData.push(['出席統計']);
                    attendanceData.push(['總學生數', totalStudents]);
                    attendanceData.push(['出席人數', presentCount]);
                    attendanceData.push(['遲到人數', lateCount]);
                    attendanceData.push(['請假人數', absentCount]);
                    attendanceData.push(['出席率', `${((presentCount / totalStudents) * 100).toFixed(1)}%`]);
                }
                
                attendanceData.push(['']);
                attendanceData.push(['匯出時間', new Date().toLocaleString('zh-TW')]);
                
                const attendanceWorksheet = XLSX.utils.aoa_to_sheet(attendanceData);
                attendanceWorksheet['!cols'] = [
                    { width: 8 }, { width: 15 }, { width: 12 }, { width: 20 }, { width: 20 }
                ];
                XLSX.utils.book_append_sheet(workbook, attendanceWorksheet, '課程出席記錄');
                
                const fileName = `課程出席記錄_${record.course}_${record.weekDate}_${new Date().toISOString().split('T')[0]}.xlsx`;
                XLSX.writeFile(workbook, fileName);
            }
            
            alert(`${record.title} 的記錄已成功匯出！`);
        }

        // 匯出所有課程記錄
        function exportAllSavedRecords() {
            if (!window.savedRecords || window.savedRecords.length === 0) {
                alert('沒有可匯出的課程記錄！');
                return;
            }
            
            const workbook = XLSX.utils.book_new();
            
            // 課程記錄匯總工作表
            const summaryData = [
                ['所有課程記錄匯總'],
                [''],
                ['序號', '課程組別', '授課教師', '年級', '上課時間', '總學生數', '出席人數', '遲到人數', '請假人數', '出席率', '建立時間', '更新時間']
            ];
            
            window.savedRecords.forEach((record, index) => {
                const totalStudents = record.students ? record.students.length : 0;
                const presentCount = record.students ? record.students.filter(s => s.attendanceStatus === '出席').length : 0;
                const lateCount = record.students ? record.students.filter(s => s.attendanceStatus === '遲到').length : 0;
                const absentCount = record.students ? record.students.filter(s => s.attendanceStatus === '請假').length : 0;
                const attendanceRate = totalStudents > 0 ? `${((presentCount / totalStudents) * 100).toFixed(1)}%` : '0%';
                
                summaryData.push([
                    index + 1,
                    record.course,
                    record.teacher,
                    record.grade,
                    `${record.weekDateText} ${record.period}`,
                    totalStudents,
                    presentCount,
                    lateCount,
                    absentCount,
                    attendanceRate,
                    record.createTime,
                    record.updateTime
                ]);
            });
            
            summaryData.push(['']);
            summaryData.push(['總課程數', window.savedRecords.length]);
            summaryData.push(['匯出時間', new Date().toLocaleString('zh-TW')]);
            
            const summaryWorksheet = XLSX.utils.aoa_to_sheet(summaryData);
            summaryWorksheet['!cols'] = [
                { width: 8 }, { width: 15 }, { width: 12 }, { width: 10 }, 
                { width: 25 }, { width: 10 }, { width: 10 }, { width: 10 }, 
                { width: 10 }, { width: 10 }, { width: 20 }, { width: 20 }
            ];
            XLSX.utils.book_append_sheet(workbook, summaryWorksheet, '課程記錄匯總');
            
            // 詳細出席記錄工作表
            const detailData = [
                ['所有課程詳細出席記錄'],
                [''],
                ['序號', '課程組別', '授課教師', '年級', '上課時間', '學生姓名', '出席狀態', '請假原因', '點名時間']
            ];
            
            let detailIndex = 1;
            window.savedRecords.forEach(record => {
                if (record.students && record.students.length > 0) {
                    record.students.forEach(student => {
                        detailData.push([
                            detailIndex++,
                            record.course,
                            record.teacher,
                            record.grade,
                            `${record.weekDateText} ${record.period}`,
                            student.studentName,
                            student.attendanceStatus,
                            student.leaveReason || '',
                            student.submitTime
                        ]);
                    });
                }
            });
            
            if (detailIndex > 1) {
                const detailWorksheet = XLSX.utils.aoa_to_sheet(detailData);
                detailWorksheet['!cols'] = [
                    { width: 8 }, { width: 15 }, { width: 12 }, { width: 10 }, 
                    { width: 25 }, { width: 15 }, { width: 12 }, { width: 20 }, { width: 20 }
                ];
                XLSX.utils.book_append_sheet(workbook, detailWorksheet, '詳細出席記錄');
            }
            
            const fileName = `課堂管理系統_所有課程記錄_${new Date().toISOString().split('T')[0]}.xlsx`;
            XLSX.writeFile(workbook, fileName);
            
            alert('所有課程記錄已成功匯出！');
        }

        // 刪除儲存記錄
        function removeSavedRecord(index) {
            if (!window.savedRecords || !window.savedRecords[index]) {
                alert('記錄不存在！');
                return;
            }
            
            const record = window.savedRecords[index];
            const recordName = record.title;
            
            if (confirm(`確定要刪除記錄「${recordName}」嗎？`)) {
                window.savedRecords.splice(index, 1);
                updateRecordManagementList();
                alert('記錄已刪除！');
            }
        }

        // 清空所有課程記錄
        function clearAllSavedRecords() {
            if (!window.savedRecords || window.savedRecords.length === 0) {
                alert('沒有可清空的課程記錄！');
                return;
            }
            
            if (confirm('確定要清空所有課程記錄嗎？此操作無法復原！')) {
                window.savedRecords = [];
                updateRecordManagementList();
                alert('所有課程記錄已清空！');
            }
        }

        // 重新整理記錄管理
        function refreshRecordManagement() {
            updateRecordManagementList();
            alert('記錄管理已重新整理！');
        }

        // 匯出所有原始記錄Excel
        function exportAllToExcel() {
            const allRecords = [...attendanceRecords, ...observationRecords];
            
            if (allRecords.length === 0) {
                alert('沒有可匯出的原始記錄！');
                return;
            }
            
            const workbook = XLSX.utils.book_new();
            
            // 點名記錄工作表
            if (attendanceRecords.length > 0) {
                const attendanceData = [
                    ['課堂點名記錄（原始資料）'],
                    [''],
                    ['序號', '學生姓名', '周次日期', '節次', '課程', '出席狀態', '請假原因', '遲到原因', '點名時間']
                ];
                
                attendanceRecords.forEach((record, index) => {
                    attendanceData.push([
                        index + 1,
                        record.studentName,
                        record.weekDateText,
                        record.period,
                        record.course,
                        record.attendanceStatus,
                        record.leaveReason || '',
                        record.lateReason || '',
                        record.submitTime
                    ]);
                });
                
                const attendanceWorksheet = XLSX.utils.aoa_to_sheet(attendanceData);
                XLSX.utils.book_append_sheet(workbook, attendanceWorksheet, '點名記錄');
            }
            
            // 觀察記錄工作表
            if (observationRecords.length > 0) {
                const observationData = [
                    ['教師觀察記錄（原始資料）'],
                    [''],
                    ['序號', '學生姓名', '周次日期', '節次', '課程', '課堂參與度', '學習態度', '作業完成情況', '行為表現', '特殊事件記錄', '送出時間']
                ];
                
                observationRecords.forEach((record, index) => {
                    observationData.push([
                        index + 1,
                        record.studentName,
                        record.weekDateText,
                        record.period,
                        record.course,
                        record.participation.join(', ') || '未填寫',
                        record.attitude.join(', ') || '未填寫',
                        record.homework.join(', ') || '未填寫',
                        record.behavior || '未填寫',
                        record.specialEvent || '未填寫',
                        record.submitTime
                    ]);
                });
                
                const observationWorksheet = XLSX.utils.aoa_to_sheet(observationData);
                XLSX.utils.book_append_sheet(workbook, observationWorksheet, '觀察記錄');
            }
            
            const fileName = `課堂管理系統原始記錄_${new Date().toISOString().split('T')[0]}.xlsx`;
            XLSX.writeFile(workbook, fileName);
            
            alert('原始記錄Excel檔案已成功匯出！');
        }

        // 初始化系統
        init();
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'96b35b1ce7374a56',t:'MTc1NDUzMzg2Ny4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
