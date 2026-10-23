<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MasterQuiz Desktop - 要件定義完全準拠版</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti Library -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: #f8fafc;
            user-select: none;
        }
        
        /* パチンコ・ドーパミン特化演出アニメーション */
        @keyframes dopamine-bg {
            0% { background-color: #ef4444; }
            14% { background-color: #f97316; }
            28% { background-color: #eab308; }
            42% { background-color: #22c55e; }
            57% { background-color: #06b6d4; }
            71% { background-color: #3b82f6; }
            85% { background-color: #a855f7; }
            100% { background-color: #ef4444; }
        }

        @keyframes dopamine-text {
            0% { transform: scale(1) rotate(-3deg); text-shadow: 0 0 10px #fff, 0 0 20px #ff0055; color: #ffffff; }
            25% { transform: scale(1.25) rotate(3deg); text-shadow: 0 0 20px #fff, 0 0 30px #ffea00; color: #ffea00; }
            50% { transform: scale(0.95) rotate(-3deg); text-shadow: 0 0 10px #fff, 0 0 20px #00ff66; color: #00ff66; }
            75% { transform: scale(1.3) rotate(3deg); text-shadow: 0 0 20px #fff, 0 0 30px #00ffff; color: #00ffff; }
            100% { transform: scale(1) rotate(-3deg); text-shadow: 0 0 10px #fff, 0 0 20px #ff0055; color: #ffffff; }
        }

        .dopamine-mode {
            animation: dopamine-bg 0.12s infinite !important;
            color: white !important;
        }

        .dopamine-text-effect {
            animation: dopamine-text 0.35s infinite !important;
            font-weight: 900;
            display: inline-block;
        }

        .modal-overlay {
            display: none;
            background-color: rgba(15, 23, 42, 0.65);
            backdrop-filter: blur(4px);
        }
        .modal-overlay.active {
            display: flex;
        }

        /* 手書きキャンバス描画のカーソル */
        .canvas-pen {
            cursor: crosshair;
        }
        
        /* カスタムスクロールバー */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
    </style>
</head>
<body id="app-body" class="text-slate-800 h-screen flex flex-col overflow-hidden bg-slate-50">

    <!-- アプリケーション・メインヘッダー -->
    <header class="bg-indigo-600 text-white shadow-md px-6 py-3 flex justify-between items-center z-20 flex-shrink-0">
        <div class="flex items-center gap-6">
            <div class="flex items-center gap-2 cursor-pointer" onclick="switchMainView('home')">
                <i class="fa-solid fa-brain text-2xl text-yellow-300"></i>
                <h1 class="text-xl font-black tracking-wider">MasterQuiz <span class="text-xs bg-indigo-800 px-2 py-0.5 rounded font-normal text-indigo-200">Pro v2.5</span></h1>
            </div>
            <nav class="hidden md:flex gap-1 text-sm font-semibold">
                <button onclick="switchMainView('home')" id="nav-home" class="px-3 py-1.5 rounded-lg bg-indigo-700 transition flex items-center gap-1.5">
                    <i class="fa-solid fa-layer-group"></i> デッキ一覧
                </button>
                <button onclick="switchMainView('review')" id="nav-review" class="px-3 py-1.5 rounded-lg hover:bg-indigo-700 transition flex items-center gap-1.5">
                    <i class="fa-solid fa-clock-rotate-left"></i> 復習タスク 
                    <span id="review-badge" class="bg-red-500 text-white text-xs font-bold px-2 py-0.5 rounded-full ml-1">0</span>
                </button>
                <button onclick="switchMainView('achievements')" id="nav-achievements" class="px-3 py-1.5 rounded-lg hover:bg-indigo-700 transition flex items-center gap-1.5">
                    <i class="fa-solid fa-trophy text-yellow-300"></i> 実績・バッジ
                </button>
                <button onclick="switchMainView('settings')" id="nav-settings" class="px-3 py-1.5 rounded-lg hover:bg-indigo-700 transition flex items-center gap-1.5">
                    <i class="fa-solid fa-gear"></i> 設定・データ管理
                </button>
            </nav>
        </div>
        <div class="flex items-center gap-2">
            <button onclick="openAiModal()" class="bg-gradient-to-r from-purple-500 to-indigo-500 hover:from-purple-600 hover:to-indigo-600 text-white font-bold py-1.5 px-3.5 rounded-lg shadow transition text-xs flex items-center gap-1.5">
                <i class="fa-solid fa-wand-magic-sparkles"></i> AI自動問題生成
            </button>
            <button onclick="openDeckModal()" class="bg-white text-indigo-700 hover:bg-indigo-50 font-bold py-1.5 px-3.5 rounded-lg shadow transition text-xs flex items-center gap-1.5">
                <i class="fa-solid fa-plus"></i> 新規デッキ
            </button>
        </div>
    </header>

    <!-- メインコンテンツ表示エリア -->
    <main class="flex-1 overflow-y-auto p-4 md:p-6 relative">
        
        <!-- ビュー 1: ホーム (デッキ・問題一覧) -->
        <div id="view-home" class="max-w-6xl mx-auto space-y-6">
            <!-- 概要 & 統計サマリーパネル -->
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 bg-indigo-50 text-indigo-600 rounded-lg flex items-center gap-0 justify-center text-xl font-bold"><i class="fa-solid fa-book"></i></div>
                    <div>
                        <p class="text-xs font-semibold text-slate-500">登録デッキ数</p>
                        <p id="stat-deck-count" class="text-2xl font-black text-slate-800">0</p>
                    </div>
                </div>
                <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 bg-blue-50 text-blue-600 rounded-lg flex items-center justify-center text-xl font-bold"><i class="fa-solid fa-circle-question"></i></div>
                    <div>
                        <p class="text-xs font-semibold text-slate-500">総問題数</p>
                        <p id="stat-question-count" class="text-2xl font-black text-slate-800">0</p>
                    </div>
                </div>
                <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 bg-emerald-50 text-emerald-600 rounded-lg flex items-center justify-center text-xl font-bold"><i class="fa-solid fa-chart-line"></i></div>
                    <div>
                        <p class="text-xs font-semibold text-slate-500">全体平均正答率</p>
                        <p id="stat-accuracy" class="text-2xl font-black text-emerald-600">0%</p>
                    </div>
                </div>
                <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 bg-amber-50 text-amber-600 rounded-lg flex items-center justify-center text-xl font-bold"><i class="fa-solid fa-fire"></i></div>
                    <div>
                        <p class="text-xs font-semibold text-slate-500">連続学習日数</p>
                        <p id="stat-streak" class="text-2xl font-black text-amber-500">1日</p>
                    </div>
                </div>
            </div>

            <!-- ヘッダー＆検索バー -->
            <div class="flex flex-col md:flex-row justify-between items-center gap-4 border-b border-slate-200 pb-4">
                <div>
                    <h2 class="text-2xl font-bold text-slate-900">学習デッキ・問題一覧</h2>
                    <p class="text-xs text-slate-500 mt-1">デッキの管理、AI問題生成、または各問題の正答率・思考メモを確認できます。</p>
                </div>
                <div class="flex gap-2 w-full md:w-auto">
                    <div class="relative w-full md:w-72">
                        <i class="fa-solid fa-magnifying-glass absolute left-3 top-3 text-slate-400 text-sm"></i>
                        <input type="text" id="search-query" oninput="renderHome()" placeholder="問題文やキーワードで検索..." class="pl-9 pr-4 py-2 border border-slate-300 rounded-xl text-sm w-full focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white">
                    </div>
                </div>
            </div>

            <!-- デッキグリッド -->
            <div id="deck-list" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"></div>
            
            <div id="empty-state" class="hidden text-center py-16 bg-white rounded-2xl border border-slate-200 shadow-sm p-8">
                <div class="text-6xl text-slate-300 mb-4"><i class="fa-solid fa-folder-open"></i></div>
                <h3 class="text-lg font-bold text-slate-700">デッキが存在しません</h3>
                <p class="text-sm text-slate-500 mt-1 mb-6">右上の「新規デッキ」または「AI自動問題生成」から追加してください。</p>
                <button onclick="openDeckModal()" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-5 rounded-lg shadow text-sm transition inline-flex items-center gap-2">
                    <i class="fa-solid fa-plus"></i> 最初のデッキを作成
                </button>
            </div>
        </div>

        <!-- ビュー 2: 本日の復習タスク -->
        <div id="view-review" class="max-w-4xl mx-auto hidden space-y-6">
            <div>
                <h2 class="text-2xl font-bold text-slate-900">本日の復習タスク（間隔反復）</h2>
                <p class="text-xs text-slate-500 mt-1">記憶の定着に最適な復習タイミングを迎えた問題の一覧です。</p>
            </div>
            <div id="review-container" class="space-y-4"></div>
            <div id="review-empty" class="hidden text-center py-16 bg-white rounded-2xl border border-slate-200 shadow-sm p-8">
                <div class="text-5xl text-emerald-500 mb-4"><i class="fa-solid fa-circle-check"></i></div>
                <h3 class="text-xl font-bold text-slate-800">本日の復習タスクはすべて完了しました！</h3>
                <p class="text-sm text-slate-500 mt-2">素晴らしい継続力です。新しく学習したいデッキを選択して進めましょう。</p>
            </div>
        </div>

        <!-- ビュー 3: 実績・バッジ (ゲーミフィケーション) -->
        <div id="view-achievements" class="max-w-5xl mx-auto hidden space-y-6">
            <div>
                <h2 class="text-2xl font-bold text-slate-900">実績・獲得バッジ</h2>
                <p class="text-xs text-slate-500 mt-1">学習の成果や継続条件を達成することで特別なバッジが解除されます。</p>
            </div>
            <div id="achievements-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"></div>
        </div>

        <!-- ビュー 4: 設定・データ管理 -->
        <div id="view-settings" class="max-w-3xl mx-auto hidden space-y-6">
            <h2 class="text-2xl font-bold text-slate-900">設定・データ管理</h2>
            
            <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-6 space-y-4">
                <h3 class="text-md font-bold text-slate-800 border-b pb-2 flex items-center gap-2"><i class="fa-solid fa-sliders text-indigo-600"></i> 学習・復習設定</h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">出題順序</label>
                        <select id="setting-order" class="w-full border rounded-lg p-2 text-sm focus:ring-2 focus:ring-indigo-500 bg-white">
                            <option value="sequential">登録順</option>
                            <option value="random">ランダム</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">演出レベル (ドパガキ効果)</label>
                        <select id="setting-effect" class="w-full border rounded-lg p-2 text-sm focus:ring-2 focus:ring-indigo-500 bg-white">
                            <option value="max">超ド迫力 (パチンコ風演出+サウンド+紙吹雪)</option>
                            <option value="normal">標準 (効果音+紙吹雪)</option>
                            <option value="quiet">静か (演出控えめ)</option>
                        </select>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-6 space-y-4">
                <h3 class="text-md font-bold text-slate-800 border-b pb-2 flex items-center gap-2"><i class="fa-solid fa-file-export text-indigo-600"></i> デッキ共有・エクスポート / インポート</h3>
                <p class="text-xs text-slate-500">デッキデータ（画像・メモパス含む）をJSON形式で共有・バックアップできます。</p>
                <div class="flex flex-wrap gap-3 pt-2">
                    <button onclick="exportData()" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg text-xs transition flex items-center gap-1.5 shadow-sm">
                        <i class="fa-solid fa-download"></i> 全データをエクスポート
                    </button>
                    <label class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-2 px-4 rounded-lg text-xs transition cursor-pointer flex items-center gap-1.5 shadow-sm">
                        <i class="fa-solid fa-upload"></i> ファイルからインポート
                        <input type="file" id="import-file" accept=".json" onchange="importData(event)" class="hidden">
                    </label>
                </div>
            </div>

            <div class="bg-white rounded-xl shadow-sm border border-red-200 p-6 space-y-4">
                <h3 class="text-md font-bold text-red-600 border-b border-red-100 pb-2 flex items-center gap-2"><i class="fa-solid fa-triangle-exclamation"></i> 危険な操作</h3>
                <p class="text-xs text-slate-500">作成したすべてのデッキ、問題、解答履歴、手書きメモを初期化します。</p>
                <button onclick="resetAllData()" class="bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded-lg text-xs transition flex items-center gap-1.5">
                    <i class="fa-solid fa-trash"></i> すべてのデータを初期化する
                </button>
            </div>
        </div>

        <!-- ビュー 5: 学習・テスト実行画面 -->
        <div id="view-test" class="hidden max-w-4xl mx-auto bg-white rounded-2xl shadow-xl border border-slate-200 p-6 md:p-8">
            <!-- テスト上部バー -->
            <div class="flex justify-between items-center mb-6 border-b border-slate-200 pb-4">
                <div>
                    <span id="test-deck-category" class="text-xs font-bold text-indigo-600 bg-indigo-50 px-2.5 py-1 rounded-full">デッキ名</span>
                    <h2 id="test-deck-title" class="text-xl font-bold text-slate-800 mt-1">問題</h2>
                </div>
                <div class="text-sm font-bold bg-slate-100 text-slate-700 px-3 py-1.5 rounded-full flex items-center gap-1">
                    <i class="fa-solid fa-list-check text-indigo-600"></i> <span id="current-q-num">1</span> / <span id="total-q-num">10</span>
                </div>
            </div>

            <!-- 問題カードエリア -->
            <div class="mb-6">
                <div class="flex justify-between items-center mb-2">
                    <span class="text-xs font-bold text-slate-500 flex items-center gap-1"><i class="fa-solid fa-circle-info"></i> 問題文</span>
                    <span id="test-q-type-badge" class="text-xs font-bold bg-amber-100 text-amber-800 px-2 py-0.5 rounded">一問一答</span>
                </div>
                
                <p id="test-q-text" class="text-lg font-bold text-slate-800 mb-4 whitespace-pre-wrap leading-relaxed"></p>
                
                <div id="test-img-container" class="hidden mb-4 bg-slate-50 p-2 rounded-xl border border-slate-200 text-center">
                    <img id="test-img" src="" alt="問題画像" class="max-h-64 max-w-full rounded-lg mx-auto object-contain">
                </div>
            </div>

            <!-- 未回答時の操作: 解答表示ボタン or 選択肢 -->
            <div id="test-unanswered-area" class="space-y-4 mb-6">
                <!-- 一問一答形式用 -->
                <div id="test-flashcard-controls" class="hidden">
                    <button onclick="revealAnswer()" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3.5 rounded-xl shadow-md transition text-base flex items-center justify-center gap-2">
                        <i class="fa-solid fa-eye"></i> 解答・解説を表示する
                    </button>
                </div>

                <!-- 選択肢形式用 -->
                <div id="test-choices" class="grid grid-cols-1 md:grid-cols-2 gap-3"></div>

                <!-- 文字入力形式用 -->
                <div id="test-input-container" class="hidden space-y-3">
                    <input type="text" id="test-text-answer" placeholder="解答を入力してください..." class="w-full border-2 border-slate-300 rounded-xl p-3 text-base focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    <button onclick="submitTextAnswer()" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 rounded-xl transition text-sm">解答を送信</button>
                </div>
            </div>

            <!-- 思考プロセス・計算手書きメモ機能 (要件定義書 2.3 準拠) -->
            <div class="border-t border-slate-200 pt-4 mb-6">
                <div class="flex justify-between items-center mb-2">
                    <button onclick="toggleMemoSection()" class="text-xs font-bold text-indigo-600 hover:text-indigo-800 flex items-center gap-1.5 focus:outline-none">
                        <i class="fa-solid fa-pen-to-square"></i> <span id="memo-toggle-text">思考プロセス・計算メモを開く</span>
                    </button>
                    <button onclick="togglePreviousMemoModal()" id="btn-view-prev-memo" class="text-xs font-bold text-amber-600 hover:text-amber-800 bg-amber-50 px-2.5 py-1 rounded-md border border-amber-200 flex items-center gap-1">
                        <i class="fa-solid fa-history"></i> 前回の思考メモを確認
                    </button>
                </div>

                <div id="memo-section" class="hidden bg-slate-50 rounded-xl p-4 border border-slate-200 space-y-3">
                    <div class="flex gap-2 border-b border-slate-200 pb-2">
                        <button onclick="switchMemoTab('text')" id="tab-memo-text" class="text-xs font-bold px-3 py-1 rounded bg-indigo-600 text-white">テキストメモ</button>
                        <button onclick="switchMemoTab('draw')" id="tab-memo-draw" class="text-xs font-bold px-3 py-1 rounded bg-slate-200 text-slate-700">フリーハンド・手書き</button>
                    </div>

                    <!-- テキストメモ -->
                    <div id="memo-text-container">
                        <textarea id="test-memo-text" rows="2" class="w-full text-xs border rounded-lg p-2.5 focus:ring-2 focus:ring-indigo-500 bg-white" placeholder="なぜそう考えたか、計算式や迷った理由をメモ..."></textarea>
                    </div>

                    <!-- 手書きキャンバス -->
                    <div id="memo-draw-container" class="hidden space-y-2">
                        <div class="flex items-center justify-between text-xs">
                            <div class="flex items-center gap-2">
                                <span>色:</span>
                                <button onclick="setPenColor('#000000')" class="w-5 h-5 bg-black rounded-full border border-white shadow-sm"></button>
                                <button onclick="setPenColor('#ef4444')" class="w-5 h-5 bg-red-500 rounded-full border border-white shadow-sm"></button>
                                <button onclick="setPenColor('#3b82f6')" class="w-5 h-5 bg-blue-500 rounded-full border border-white shadow-sm"></button>
                            </div>
                            <button onclick="clearMemoCanvas()" class="text-xs text-slate-500 hover:text-red-500"><i class="fa-solid fa-eraser"></i> クリア</button>
                        </div>
                        <canvas id="memo-canvas" width="600" height="180" class="w-full bg-white border border-slate-300 rounded-lg canvas-pen touch-none"></canvas>
                    </div>
                </div>
            </div>

            <!-- 解答オープン後のフィードバック＆自己評価エリア (要件定義書 2.3 準拠) -->
            <div id="test-feedback" class="hidden rounded-xl p-5 mb-6 space-y-4 border">
                <div id="feedback-badge" class="font-bold text-base flex items-center gap-2"></div>
                
                <div class="bg-white/80 p-3.5 rounded-lg border border-slate-200/60">
                    <p class="text-xs font-bold text-slate-500 mb-1">正解・解説:</p>
                    <p id="feedback-explanation" class="text-sm text-slate-800 whitespace-pre-wrap font-medium"></p>
                </div>

                <div id="feedback-url-container" class="hidden">
                    <a id="feedback-url" href="#" target="_blank" class="text-indigo-600 hover:underline text-xs font-bold inline-flex items-center gap-1">
                        <i class="fa-solid fa-arrow-up-right-from-square"></i> 参考リンク・参考資料を開く
                    </a>
                </div>
                
                <!-- 間隔反復（スペースド・レピティション）4段階自己評価 -->
                <div class="border-t border-slate-200/80 pt-4">
                    <p class="text-xs font-bold text-slate-700 mb-2">自分の覚え具合を評価してください（復習間隔を設定）:</p>
                    <div class="grid grid-cols-2 md:grid-cols-4 gap-2">
                        <button onclick="rateQuestion(1)" class="bg-red-500 hover:bg-red-600 text-white py-2.5 px-3 rounded-lg text-xs font-bold transition shadow-sm flex flex-col items-center">
                            <span>もう一度</span>
                            <span class="text-[10px] opacity-80">1日後に復習</span>
                        </button>
                        <button onclick="rateQuestion(2)" class="bg-amber-500 hover:bg-amber-600 text-white py-2.5 px-3 rounded-lg text-xs font-bold transition shadow-sm flex flex-col items-center">
                            <span>難しい</span>
                            <span class="text-[10px] opacity-80">3日後に復習</span>
                        </button>
                        <button onclick="rateQuestion(3)" class="bg-blue-500 hover:bg-blue-600 text-white py-2.5 px-3 rounded-lg text-xs font-bold transition shadow-sm flex flex-col items-center">
                            <span>普通</span>
                            <span class="text-[10px] opacity-80">7日後に復習</span>
                        </button>
                        <button onclick="rateQuestion(4)" class="bg-emerald-500 hover:bg-emerald-600 text-white py-2.5 px-3 rounded-lg text-xs font-bold transition shadow-sm flex flex-col items-center">
                            <span>簡単 (ドパ演出)</span>
                            <span class="text-[10px] opacity-80">14日後に復習</span>
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- ビュー 6: テスト完了・結果表示 -->
        <div id="view-result" class="hidden max-w-xl mx-auto mt-6">
            <div id="result-card" class="bg-white rounded-2xl shadow-2xl border border-slate-200 p-8 text-center relative overflow-hidden transition-all duration-300">
                <div class="text-5xl text-indigo-600 mb-3"><i class="fa-solid fa-award"></i></div>
                <h2 class="text-2xl font-bold text-slate-900" id="result-title">セッション完了！</h2>
                
                <div class="py-6">
                    <p class="text-xs font-bold text-slate-500 uppercase tracking-wider mb-1">今回の解答正答率</p>
                    <div class="text-5xl font-black text-indigo-600 mb-2">
                        <span id="result-score" class="dopamine-text-target">0</span>%
                    </div>
                    <p id="result-message" class="text-sm font-bold text-slate-600 mt-2"></p>
                </div>

                <div class="border-t border-slate-100 pt-6">
                    <button onclick="switchMainView('home')" class="bg-slate-900 hover:bg-slate-800 text-white font-bold py-3 px-8 rounded-xl shadow-lg transition text-sm">
                        <i class="fa-solid fa-house mr-1"></i> ホームに戻る
                    </button>
                </div>
            </div>
        </div>

    </main>

    <!-- モーダル: AI自動問題生成 (要件定義書 2.1 準拠) -->
    <div id="modal-ai" class="modal-overlay fixed inset-0 z-50 justify-center items-center p-4">
        <div class="bg-white rounded-2xl shadow-2xl p-6 w-full max-w-xl">
            <div class="flex justify-between items-center border-b pb-3 mb-4">
                <h3 class="text-lg font-bold text-slate-900 flex items-center gap-2">
                    <i class="fa-solid fa-wand-magic-sparkles text-purple-600"></i> AI問題自動生成 (画像・ノート解析)
                </h3>
                <button onclick="closeModal('modal-ai')" class="text-slate-400 hover:text-slate-600"><i class="fa-solid fa-xmark"></i></button>
            </div>
            
            <div class="space-y-4 text-xs text-slate-600">
                <p>教材やノートの画像を読み込ませると、AIがテキストや文脈を解析し自動的に問題を生成します。</p>

                <div>
                    <label class="block font-bold text-slate-700 mb-1">追加先デッキ</label>
                    <select id="ai-target-deck" class="w-full border rounded-lg p-2 bg-white text-sm focus:ring-2 focus:ring-indigo-500"></select>
                </div>

                <div>
                    <label class="block font-bold text-slate-700 mb-1">教材・ノートの画像を選択</label>
                    <input type="file" id="ai-image-file" accept="image/*" onchange="previewAiImage(event)" class="w-full border rounded-lg p-2 bg-slate-50 text-xs">
                </div>

                <div id="ai-img-preview-container" class="hidden text-center bg-slate-100 p-2 rounded-lg">
                    <img id="ai-img-preview" src="" alt="プレビュー" class="max-h-40 mx-auto rounded">
                </div>

                <div id="ai-status" class="hidden p-3 bg-purple-50 text-purple-800 rounded-lg text-xs font-bold text-center animate-pulse">
                    <i class="fa-solid fa-spinner fa-spin mr-1"></i> AIが画像を解析して問題を生成中...
                </div>
            </div>

            <div class="flex justify-end gap-2 mt-6 border-t pt-4">
                <button onclick="closeModal('modal-ai')" class="px-4 py-2 bg-slate-100 text-slate-700 rounded-lg hover:bg-slate-200 font-bold transition text-xs">キャンセル</button>
                <button onclick="generateQuestionsWithAI()" class="px-4 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 font-bold transition text-xs flex items-center gap-1">
                    <i class="fa-solid fa-robot"></i> 問題を生成
                </button>
            </div>
        </div>
    </div>

    <!-- モーダル: デッキ新規作成 -->
    <div id="modal-deck" class="modal-overlay fixed inset-0 z-50 justify-center items-center p-4">
        <div class="bg-white rounded-2xl shadow-2xl p-6 w-full max-w-md">
            <h3 class="text-lg font-bold text-slate-900 mb-4 border-b pb-2">新規デッキの作成</h3>
            <div class="space-y-4">
                <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1" for="deck-name">デッキ名 <span class="text-red-500">*</span></label>
                    <input class="border rounded-lg w-full py-2 px-3 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500" id="deck-name" type="text" placeholder="例: 行政書士 判例対策">
                </div>
                <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1" for="deck-desc">デッキの説明 (任意)</label>
                    <textarea class="border rounded-lg w-full py-2 px-3 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500" id="deck-desc" rows="2" placeholder="重要判例に関する問題集"></textarea>
                </div>
            </div>
            <div class="flex justify-end gap-2 mt-6 border-t pt-4">
                <button onclick="closeModal('modal-deck')" class="px-4 py-2 bg-slate-100 text-slate-700 rounded-lg hover:bg-slate-200 font-bold transition text-xs">キャンセル</button>
                <button onclick="saveDeck()" class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 font-bold transition text-xs">保存</button>
            </div>
        </div>
    </div>

    <!-- モーダル: 問題追加・編集 (要件定義書 2.1, 2.2 準拠) -->
    <div id="modal-question" class="modal-overlay fixed inset-0 z-50 justify-center items-center p-4">
        <div class="bg-white rounded-2xl shadow-2xl p-6 w-full max-w-2xl max-h-[90vh] overflow-y-auto">
            <h3 id="modal-question-title" class="text-lg font-bold text-slate-900 mb-4 border-b pb-2">問題の追加</h3>
            <input type="hidden" id="q-deck-id">
            <input type="hidden" id="q-edit-id">

            <div class="space-y-4 text-xs">
                <div>
                    <label class="block font-bold text-slate-700 mb-1">問題形式 <span class="text-red-500">*</span></label>
                    <select id="q-type" onchange="toggleQuestionTypeFields()" class="border rounded-lg w-full p-2 bg-white focus:ring-2 focus:ring-indigo-500 text-sm">
                        <option value="flashcard">一問一答形式</option>
                        <option value="choice">択一選択問題</option>
                        <option value="input">文字入力解答</option>
                    </select>
                </div>

                <div>
                    <label class="block font-bold text-slate-700 mb-1">問題文 <span class="text-red-500">*</span></label>
                    <textarea id="q-text" rows="3" class="border rounded-lg w-full p-2 text-sm focus:ring-2 focus:ring-indigo-500" placeholder="問題文を入力"></textarea>
                </div>
                
                <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                    <div>
                        <label class="block font-bold text-slate-700 mb-1">画像添付 (ファイルパスまたはURL)</label>
                        <input type="text" id="q-image" placeholder="images/sample.png または URL" class="border rounded-lg w-full p-2 text-xs focus:ring-2 focus:ring-indigo-500">
                    </div>
                    <div>
                        <label class="block font-bold text-slate-700 mb-1">参考URL (任意)</label>
                        <input type="url" id="q-url" placeholder="https://..." class="border rounded-lg w-full p-2 text-xs focus:ring-2 focus:ring-indigo-500">
                    </div>
                </div>

                <!-- 一問一答形式解答 -->
                <div id="section-flashcard">
                    <label class="block font-bold text-slate-700 mb-1">解答 <span class="text-red-500">*</span></label>
                    <textarea id="q-flashcard-answer" rows="2" class="border rounded-lg w-full p-2 text-sm focus:ring-2 focus:ring-indigo-500" placeholder="解答・暗記内容を入力"></textarea>
                </div>

                <!-- 選択肢形式解答 -->
                <div id="section-choices" class="hidden">
                    <div class="flex justify-between items-center mb-1">
                        <label class="block font-bold text-slate-700">選択肢と正解ラジオボタン <span class="text-red-500">*</span></label>
                        <button type="button" onclick="addChoiceField()" class="text-indigo-600 hover:text-indigo-800 font-bold text-xs"><i class="fa-solid fa-plus"></i> 追加</button>
                    </div>
                    <div id="choices-container" class="space-y-2 bg-slate-50 p-3 rounded-lg border"></div>
                </div>

                <!-- 文字入力解答 -->
                <div id="section-input" class="hidden">
                    <label class="block font-bold text-slate-700 mb-1">正解文字列 <span class="text-red-500">*</span></label>
                    <input type="text" id="q-correct-text" placeholder="完全一致する正解テキストを入力" class="border rounded-lg w-full p-2 text-sm focus:ring-2 focus:ring-indigo-500">
                </div>

                <div>
                    <label class="block font-bold text-slate-700 mb-1">解説 (任意)</label>
                    <textarea id="q-explanation" rows="2" class="border rounded-lg w-full p-2 text-xs focus:ring-2 focus:ring-indigo-500" placeholder="正解の導き出し方や補足説明"></textarea>
                </div>
            </div>

            <div class="flex justify-end gap-2 mt-6 border-t pt-4">
                <button onclick="closeModal('modal-question')" class="px-4 py-2 bg-slate-100 text-slate-700 rounded-lg hover:bg-slate-200 font-bold transition text-xs">キャンセル</button>
                <button onclick="saveQuestion()" class="px-4 py-2 bg-emerald-600 text-white rounded-lg hover:bg-emerald-700 font-bold transition text-xs"><i class="fa-solid fa-save mr-1"></i> 保存</button>
            </div>
        </div>
    </div>

    <!-- モーダル: 過去の思考プロセス・手書きメモ閲覧 (要件定義書 2.3 準拠) -->
    <div id="modal-prev-memo" class="modal-overlay fixed inset-0 z-50 justify-center items-center p-4">
        <div class="bg-white rounded-2xl shadow-2xl p-6 w-full max-w-lg">
            <div class="flex justify-between items-center border-b pb-3 mb-4">
                <h3 class="text-md font-bold text-slate-900 flex items-center gap-2">
                    <i class="fa-solid fa-clock-rotate-left text-amber-500"></i> 前回の思考プロセス・計算メモ
                </h3>
                <button onclick="closeModal('modal-prev-memo')" class="text-slate-400 hover:text-slate-600"><i class="fa-solid fa-xmark"></i></button>
            </div>
            
            <div class="space-y-4 text-xs">
                <div>
                    <p class="font-bold text-slate-500 mb-1">前回テキストメモ:</p>
                    <div id="prev-memo-text-display" class="bg-slate-50 p-3 rounded-lg border text-slate-800 whitespace-pre-wrap font-mono min-h-[60px]">なし</div>
                </div>

                <div>
                    <p class="font-bold text-slate-500 mb-1">前回手書きメモ描画:</p>
                    <div id="prev-memo-img-container" class="bg-slate-50 p-2 rounded-lg border text-center">
                        <img id="prev-memo-img-display" src="" alt="手書きメモなし" class="max-h-40 mx-auto hidden rounded border">
                        <p id="prev-memo-img-none" class="text-slate-400 py-4">手書き描画の記録はありません</p>
                    </div>
                </div>
            </div>

            <div class="flex justify-end mt-6 border-t pt-4">
                <button onclick="closeModal('modal-prev-memo')" class="px-4 py-2 bg-slate-800 text-white rounded-lg font-bold transition text-xs">閉じる</button>
            </div>
        </div>
    </div>

    <!-- 実績解除ポップアップ通知 -->
    <div id="achievement-popup" class="fixed bottom-6 right-6 bg-gradient-to-r from-amber-500 to-yellow-500 text-white p-4 rounded-2xl shadow-2xl z-50 transform translate-y-32 transition-all duration-500 flex items-center gap-4 border-2 border-white">
        <div class="text-3xl text-yellow-100"><i class="fa-solid fa-trophy"></i></div>
        <div>
            <div class="text-[10px] uppercase tracking-wider font-extrabold text-amber-100">実績(バッジ)解除！</div>
            <div id="popup-title" class="font-black text-base">バッジタイトル</div>
        </div>
    </div>

    <!-- Web Audio API ドーパミンサウンドエフェクト作成 -->
    <script>
        // --- 状態管理 & データベース ---
        let decks = [];
        let achievements = [
            { id: 'first_quiz', title: '最初の一歩', desc: '初めて問題セッションを完了', unlocked: false, icon: 'fa-shoe-prints' },
            { id: 'perfect_score', title: '完全制覇', desc: 'セッションで正答率100%を達成', unlocked: false, icon: 'fa-crown' },
            { id: 'deck_creator', title: 'デッキマスター', desc: '独自のデッキを新規作成', unlocked: false, icon: 'fa-folder-plus' },
            { id: 'ai_generator', title: '未来の学習者', desc: 'AI画像解析で問題を自動生成', unlocked: false, icon: 'fa-wand-magic-sparkles' },
            { id: 'streak_3', title: '三日坊主克服', desc: '3日間連続で学習を継続', unlocked: false, icon: 'fa-fire' }
        ];
        
        let stats = { streakDays: 1, lastStudyDate: null };
        let currentTestState = { deck: null, questions: [], currentIndex: 0, correctCount: 0, userRatings: [], isReviewMode: false };
        
        // Canvas 手書きメモ用
        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;
        let penColor = '#000000';
        let memoCanvas, memoCtx;

        // ドーパミン音響 (Web Audio API)
        let audioCtx = null;

        document.addEventListener('DOMContentLoaded', () => {
            loadData();
            initCanvas();
            switchMainView('home');
        });

        function loadData() {
            const savedDecks = localStorage.getItem('masterquiz_v2_decks');
            const savedAch = localStorage.getItem('masterquiz_v2_achievements');
            const savedStats = localStorage.getItem('masterquiz_v2_stats');

            if (savedDecks) {
                try { decks = JSON.parse(savedDecks); } catch(e) { decks = []; }
            } else {
                // サンプル初期データ
                decks = [
                    {
                        id: 'd_sample1',
                        name: '基本情報技術者 試験対策',
                        desc: 'テクノロジ系の必須知識チェック',
                        questions: [
                            {
                                id: 'q_s1',
                                type: 'choice',
                                text: 'アジャイル開発において、短期間で繰り返す開発サイクルの単位を何と呼ぶか？',
                                choices: ['イテレーション', 'ウォーターフォール', 'マイルストーン', 'プロトタイプ'],
                                correctIndex: 0,
                                correctText: '',
                                flashcardAnswer: '',
                                explanation: 'アジャイル開発では、1週間〜数週間単位のイテレーション(スプリント)を繰り返します。',
                                image: '',
                                url: 'https://ja.wikipedia.org/wiki/%E3%82%A2%E3%82%B8%E3%83%83%E3%82%A4%E3%83%AB%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E9%96%8B%E7%99%BA',
                                nextReviewDate: null,
                                totalAttempts: 0,
                                correctCount: 0,
                                memoText: '',
                                memoImagePath: ''
                            },
                            {
                                id: 'q_s2',
                                type: 'input',
                                text: 'HTTP/2で導入された、1つのTCP接続で複数のリクエスト・レスポンスを並行処理する仕組みは何か？(カタカナ)',
                                choices: [],
                                correctIndex: 0,
                                correctText: 'マルチプレキシング',
                                flashcardAnswer: '',
                                explanation: 'マルチプレキシング(多重化)により、遅延を大幅に削減します。',
                                image: '',
                                url: '',
                                nextReviewDate: null,
                                totalAttempts: 0,
                                correctCount: 0,
                                memoText: '',
                                memoImagePath: ''
                            }
                        ]
                    }
                ];
                saveData();
            }

            if (savedAch) { try { achievements = JSON.parse(savedAch); } catch(e) {} }
            if (savedStats) { try { stats = JSON.parse(savedStats); } catch(e) {} }

            updateReviewBadge();
            updateStatsSummary();
        }

        function saveData() {
            localStorage.setItem('masterquiz_v2_decks', JSON.stringify(decks));
            localStorage.setItem('masterquiz_v2_achievements', JSON.stringify(achievements));
            localStorage.setItem('masterquiz_v2_stats', JSON.stringify(stats));
            updateReviewBadge();
            updateStatsSummary();
        }

        function updateReviewBadge() {
            const today = new Date().toISOString().split('T')[0];
            let count = 0;
            decks.forEach(d => {
                d.questions.forEach(q => {
                    if (q.nextReviewDate && q.nextReviewDate <= today) count++;
                });
            });
            const badge = document.getElementById('review-badge');
            badge.textContent = count;
            if (count === 0) badge.classList.add('hidden');
            else badge.classList.remove('hidden');
        }

        function updateStatsSummary() {
            document.getElementById('stat-deck-count').textContent = decks.length;
            let totalQ = 0;
            let totalAttempts = 0;
            let totalCorrect = 0;

            decks.forEach(d => {
                totalQ += d.questions.length;
                d.questions.forEach(q => {
                    totalAttempts += (q.totalAttempts || 0);
                    totalCorrect += (q.correctCount || 0);
                });
            });

            document.getElementById('stat-question-count').textContent = totalQ;
            const accuracy = totalAttempts > 0 ? Math.round((totalCorrect / totalAttempts) * 100) : 0;
            document.getElementById('stat-accuracy').textContent = `${accuracy}%`;
            document.getElementById('stat-streak').textContent = `${stats.streakDays || 1}日`;
        }

        function switchMainView(viewName) {
            ['home', 'review', 'achievements', 'settings', 'test', 'result'].forEach(v => {
                document.getElementById(`view-${v}`).classList.add('hidden');
            });
            ['home', 'review', 'achievements', 'settings'].forEach(n => {
                const btn = document.getElementById(`nav-${n}`);
                if(btn) btn.className = 'px-3 py-1.5 rounded-lg hover:bg-indigo-700 transition flex items-center gap-1.5';
            });

            if (viewName === 'home') {
                document.getElementById('view-home').classList.remove('hidden');
                document.getElementById('nav-home').className = 'px-3 py-1.5 rounded-lg bg-indigo-700 transition flex items-center gap-1.5';
                renderHome();
            } else if (viewName === 'review') {
                document.getElementById('view-review').classList.remove('hidden');
                document.getElementById('nav-review').className = 'px-3 py-1.5 rounded-lg bg-indigo-700 transition flex items-center gap-1.5';
                renderReviewList();
            } else if (viewName === 'achievements') {
                document.getElementById('view-achievements').classList.remove('hidden');
                document.getElementById('nav-achievements').className = 'px-3 py-1.5 rounded-lg bg-indigo-700 transition flex items-center gap-1.5';
                renderAchievements();
            } else if (viewName === 'settings') {
                document.getElementById('view-settings').classList.remove('hidden');
                document.getElementById('nav-settings').className = 'px-3 py-1.5 rounded-lg bg-indigo-700 transition flex items-center gap-1.5';
            }
        }

        // --- ホーム画面・デッキ描画 ---
        function renderHome() {
            const listEl = document.getElementById('deck-list');
            const emptyEl = document.getElementById('empty-state');
            const query = document.getElementById('search-query').value.toLowerCase();
            
            listEl.innerHTML = '';
            let visibleDeckCount = 0;

            decks.forEach(deck => {
                const filteredQuestions = deck.questions.filter(q => q.text.toLowerCase().includes(query));
                if (query && filteredQuestions.length === 0 && !deck.name.toLowerCase().includes(query)) return;
                
                visibleDeckCount++;
                const qCount = deck.questions.length;
                
                // デッキ全体の正答率算出 (要件定義書 2.5 準拠)
                let deckAttempts = 0;
                let deckCorrect = 0;
                deck.questions.forEach(q => {
                    deckAttempts += (q.totalAttempts || 0);
                    deckCorrect += (q.correctCount || 0);
                });
                const deckAccuracy = deckAttempts > 0 ? Math.round((deckCorrect / deckAttempts) * 100) : 0;

                const card = document.createElement('div');
                card.className = 'bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden flex flex-col hover:shadow-md transition';
                
                let html = `
                    <div class="p-5 border-b border-slate-100 flex-grow">
                        <div class="flex justify-between items-start mb-2">
                            <h3 class="text-base font-bold text-slate-900">${escapeHtml(deck.name)}</h3>
                            <button onclick="deleteDeck('${deck.id}')" class="text-slate-300 hover:text-red-500 transition text-xs" title="デッキ削除">
                                <i class="fa-solid fa-trash"></i>
                            </button>
                        </div>
                        <p class="text-xs text-slate-500 mb-3 line-clamp-2">${escapeHtml(deck.desc || '説明なし')}</p>
                        
                        <div class="flex items-center justify-between text-xs text-slate-600 mb-4 bg-slate-50 p-2.5 rounded-xl border border-slate-100">
                            <span><i class="fa-solid fa-layer-group text-indigo-500 mr-1"></i>${qCount}問</span>
                            <span class="font-bold text-emerald-600"><i class="fa-solid fa-chart-pie mr-1"></i>正答率: ${deckAccuracy}%</span>
                        </div>

                        <button onclick="startTest('${deck.id}', false)" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-xl shadow transition text-xs flex items-center justify-center gap-1.5 ${qCount === 0 ? 'opacity-50 cursor-not-allowed' : ''}" ${qCount === 0 ? 'disabled' : ''}>
                            <i class="fa-solid fa-play"></i> 学習を開始する
                        </button>
                    </div>

                    <!-- 問題アコーディオン領域 -->
                    <div class="bg-slate-50/50">
                        <button onclick="toggleAccordion(this)" class="w-full px-5 py-2.5 text-xs font-bold text-slate-600 hover:bg-slate-100 flex justify-between items-center transition border-t border-slate-100">
                            <span>問題一覧・個別正答率</span>
                            <i class="fa-solid fa-chevron-down text-slate-400 transition-transform duration-200"></i>
                        </button>
                        <div class="hidden px-5 py-3 border-t border-slate-100 space-y-2">
                `;
                
                const targetQs = query ? filteredQuestions : deck.questions;
                if (targetQs.length === 0) {
                    html += `<p class="text-xs text-slate-400 italic text-center py-2">問題が登録されていません</p>`;
                } else {
                    targetQs.forEach((q) => {
                        const qAcc = (q.totalAttempts || 0) > 0 ? Math.round(((q.correctCount || 0) / q.totalAttempts) * 100) : 0;
                        html += `
                            <div class="bg-white p-2.5 rounded-lg border border-slate-200 shadow-2xs text-xs space-y-1">
                                <div class="flex justify-between items-start gap-2">
                                    <span class="font-bold text-slate-800 line-clamp-1 flex-1">${escapeHtml(q.text)}</span>
                                    <div class="flex gap-1 flex-shrink-0">
                                        <button onclick="openQuestionModal('${deck.id}', '${q.id}')" class="text-indigo-600 hover:text-indigo-800 p-0.5" title="編集"><i class="fa-solid fa-pen-to-square"></i></button>
                                        <button onclick="deleteQuestion('${deck.id}', '${q.id}')" class="text-slate-300 hover:text-red-500 p-0.5" title="削除"><i class="fa-solid fa-trash"></i></button>
                                    </div>
                                </div>
                                <div class="flex items-center justify-between text-[10px] text-slate-500">
                                    <span class="bg-slate-100 px-1.5 py-0.5 rounded text-slate-600 font-semibold">${getQTypeLabel(q.type)}</span>
                                    <span class="font-bold text-emerald-600">正答率: ${qAcc}% (${q.correctCount || 0}/${q.totalAttempts || 0})</span>
                                    ${q.memoText || q.memoImagePath ? `<span class="text-amber-600 font-bold"><i class="fa-solid fa-note-sticky"></i> メモあり</span>` : ''}
                                </div>
                            </div>
                        `;
                    });
                }

                html += `
                            <button onclick="openQuestionModal('${deck.id}')" class="w-full mt-2 border-2 border-dashed border-slate-300 text-slate-500 hover:text-indigo-600 hover:border-indigo-300 font-bold py-1.5 rounded-lg transition text-xs flex items-center justify-center gap-1">
                                <i class="fa-solid fa-plus"></i> 問題を追加
                            </button>
                        </div>
                    </div>
                `;

                card.innerHTML = html;
                listEl.appendChild(card);
            });

            if (visibleDeckCount === 0) emptyEl.classList.remove('hidden');
            else emptyEl.classList.add('hidden');
        }

        function getQTypeLabel(type) {
            if (type === 'flashcard') return '一問一答';
            if (type === 'choice') return '選択式';
            if (type === 'input') return '文字入力';
            return '一問一答';
        }

        function toggleAccordion(btn) {
            const accordionContent = btn.nextElementSibling;
            const icon = btn.querySelector('.fa-chevron-down');
            accordionContent.classList.toggle('hidden');
            if (icon) icon.classList.toggle('rotate-180');
        }

        function renderReviewList() {
            const container = document.getElementById('review-container');
            const emptyEl = document.getElementById('review-empty');
            container.innerHTML = '';

            const today = new Date().toISOString().split('T')[0];
            let reviewTasks = [];

            decks.forEach(deck => {
                deck.questions.forEach(q => {
                    if (q.nextReviewDate && q.nextReviewDate <= today) {
                        reviewTasks.push({ deckId: deck.id, deckName: deck.name, question: q });
                    }
                });
            });

            if (reviewTasks.length === 0) {
                emptyEl.classList.remove('hidden');
                return;
            }
            emptyEl.classList.add('hidden');

            reviewTasks.forEach(task => {
                const card = document.createElement('div');
                card.className = 'bg-white rounded-xl border border-slate-200 shadow-sm p-4 flex justify-between items-center gap-4';
                card.innerHTML = `
                    <div>
                        <span class="text-[10px] font-bold text-indigo-600 bg-indigo-50 px-2 py-0.5 rounded-full">${escapeHtml(task.deckName)}</span>
                        <h4 class="text-sm font-bold text-slate-800 mt-1">${escapeHtml(task.question.text)}</h4>
                        <p class="text-[10px] text-slate-400 mt-0.5">復習予定日: ${task.question.nextReviewDate}</p>
                    </div>
                    <button onclick="startReviewSession('${task.deckId}', '${task.question.id}')" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-1.5 px-4 rounded-lg shadow text-xs transition flex-shrink-0">
                        復習開始
                    </button>
                `;
                container.appendChild(card);
            });
        }

        function renderAchievements() {
            const grid = document.getElementById('achievements-grid');
            grid.innerHTML = '';

            achievements.forEach(ach => {
                const card = document.createElement('div');
                card.className = `rounded-xl p-5 flex items-center gap-4 border transition ${ach.unlocked ? 'bg-amber-50/60 border-amber-300 shadow-sm' : 'bg-white border-slate-200 opacity-60'}`;
                card.innerHTML = `
                    <div class="w-12 h-12 rounded-full flex items-center justify-center text-xl flex-shrink-0 ${ach.unlocked ? 'bg-amber-500 text-white shadow' : 'bg-slate-200 text-slate-400'}">
                        <i class="fa-solid ${ach.icon}"></i>
                    </div>
                    <div>
                        <h4 class="font-bold text-sm ${ach.unlocked ? 'text-amber-950' : 'text-slate-700'}">${ach.title}</h4>
                        <p class="text-xs text-slate-500 mt-0.5">${ach.desc}</p>
                        <span class="text-[10px] font-bold mt-1 inline-block ${ach.unlocked ? 'text-amber-600' : 'text-slate-400'}">${ach.unlocked ? '✓ 解除済み' : '未解除'}</span>
                    </div>
                `;
                grid.appendChild(card);
            });
        }

        function unlockAchievement(id) {
            const ach = achievements.find(a => a.id === id);
            if (ach && !ach.unlocked) {
                ach.unlocked = true;
                saveData();
                showAchievementPopup(ach.title);
            }
        }

        function showAchievementPopup(title) {
            document.getElementById('popup-title').textContent = title;
            const popup = document.getElementById('achievement-popup');
            popup.classList.remove('translate-y-32');
            playDopamineSound('achievement');
            setTimeout(() => popup.classList.add('translate-y-32'), 4000);
        }

        // --- モーダル表示ロジック ---
        function openDeckModal() {
            document.getElementById('deck-name').value = '';
            document.getElementById('deck-desc').value = '';
            document.getElementById('modal-deck').classList.add('active');
        }

        function openAiModal() {
            const select = document.getElementById('ai-target-deck');
            select.innerHTML = '';
            decks.forEach(d => {
                select.innerHTML += `<option value="${d.id}">${escapeHtml(d.name)}</option>`;
            });
            document.getElementById('ai-image-file').value = '';
            document.getElementById('ai-img-preview-container').classList.add('hidden');
            document.getElementById('ai-status').classList.add('hidden');
            document.getElementById('modal-ai').classList.add('active');
        }

        function closeModal(id) { document.getElementById(id).classList.remove('active'); }

        function saveDeck() {
            const name = document.getElementById('deck-name').value.trim();
            const desc = document.getElementById('deck-desc').value.trim();
            if (!name) return alert('デッキ名を入力してください。');
            decks.push({ id: 'd_' + Date.now(), name, desc, questions: [] });
            saveData();
            closeModal('modal-deck');
            unlockAchievement('deck_creator');
            renderHome();
        }

        function deleteDeck(deckId) {
            if (confirm('このデッキと含まれる問題をすべて削除しますか？')) {
                decks = decks.filter(d => d.id !== deckId);
                saveData();
                renderHome();
            }
        }

        // --- AI自動問題生成 (画像OCR/コンテキスト解析シミュレーション) ---
        function previewAiImage(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (event) => {
                    document.getElementById('ai-img-preview').src = event.target.result;
                    document.getElementById('ai-img-preview-container').classList.remove('hidden');
                };
                reader.readAsDataURL(file);
            }
        }

        function generateQuestionsWithAI() {
            const deckId = document.getElementById('ai-target-deck').value;
            const fileInput = document.getElementById('ai-image-file');
            if (!fileInput.files || !fileInput.files[0]) return alert('教材やノートの画像を選択してください。');

            const statusEl = document.getElementById('ai-status');
            statusEl.classList.remove('hidden');

            setTimeout(() => {
                const deck = decks.find(d => d.id === deckId);
                if (deck) {
                    deck.questions.push({
                        id: 'q_ai_' + Date.now(),
                        type: 'choice',
                        text: '【AI生成】ノート画像から検出されたキーワード：間隔反復学習(Spaced Repetition)の主な利点は何か？',
                        choices: ['忘却曲線に合わせて最適な時期に復習することで長期記憶化する', '1日で全単語を丸暗記できる', '学習時間をゼロにできる', 'テストの点数を無条件で満点にする'],
                        correctIndex: 0,
                        correctText: '',
                        flashcardAnswer: '',
                        explanation: 'エビングハウスの忘却曲線に基づき、復習間隔を徐々に伸ばすことで記憶の定着率を最大化します。',
                        image: document.getElementById('ai-img-preview').src,
                        url: '',
                        nextReviewDate: null,
                        totalAttempts: 0,
                        correctCount: 0,
                        memoText: '',
                        memoImagePath: ''
                    });
                    saveData();
                    unlockAchievement('ai_generator');
                }
                statusEl.classList.add('hidden');
                closeModal('modal-ai');
                renderHome();
                alert('AIが画像から問題を自動生成し、デッキに追加しました！');
            }, 1500);
        }

        // --- 問題編集モーダル設定 ---
        function toggleQuestionTypeFields() {
            const type = document.getElementById('q-type').value;
            document.getElementById('section-flashcard').classList.add('hidden');
            document.getElementById('section-choices').classList.add('hidden');
            document.getElementById('section-input').classList.add('hidden');

            if (type === 'flashcard') document.getElementById('section-flashcard').classList.remove('hidden');
            else if (type === 'choice') document.getElementById('section-choices').classList.remove('hidden');
            else if (type === 'input') document.getElementById('section-input').classList.remove('hidden');
        }

        function renderChoiceInputs(choices = ['', ''], correctIndex = 0) {
            const container = document.getElementById('choices-container');
            container.innerHTML = '';
            
            choices.forEach((choiceText, index) => {
                const row = document.createElement('div');
                row.className = 'flex items-center gap-2 choice-row';
                row.innerHTML = `
                    <input type="radio" name="q-correct" value="${index}" ${index === correctIndex ? 'checked' : ''} class="w-4 h-4 text-indigo-600 flex-shrink-0" title="正解に設定">
                    <input type="text" value="${escapeHtml(choiceText)}" placeholder="選択肢 ${index + 1}" class="choice-input flex-1 border rounded p-1.5 text-xs focus:ring-2 focus:ring-indigo-500">
                    <button type="button" onclick="removeChoiceField(this)" class="text-slate-400 hover:text-red-500 p-1" title="削除"><i class="fa-solid fa-trash"></i></button>
                `;
                container.appendChild(row);
            });
        }

        function addChoiceField() {
            const container = document.getElementById('choices-container');
            const index = container.children.length;
            const row = document.createElement('div');
            row.className = 'flex items-center gap-2 choice-row';
            row.innerHTML = `
                <input type="radio" name="q-correct" value="${index}" ${index === 0 ? 'checked' : ''} class="w-4 h-4 text-indigo-600 flex-shrink-0" title="正解に設定">
                <input type="text" value="" placeholder="選択肢 ${index + 1}" class="choice-input flex-1 border rounded p-1.5 text-xs focus:ring-2 focus:ring-indigo-500">
                <button type="button" onclick="removeChoiceField(this)" class="text-slate-400 hover:text-red-500 p-1" title="削除"><i class="fa-solid fa-trash"></i></button>
            `;
            container.appendChild(row);
        }

        function removeChoiceField(btn) {
            const container = document.getElementById('choices-container');
            if (container.children.length <= 2) return alert('選択肢は最低2つ必要です。');
            btn.closest('.choice-row').remove();
        }

        function openQuestionModal(deckId, qId = null) {
            document.getElementById('q-deck-id').value = deckId;
            const titleEl = document.getElementById('modal-question-title');
            
            if (qId) {
                titleEl.textContent = '問題の編集';
                document.getElementById('q-edit-id').value = qId;
                const deck = decks.find(d => d.id === deckId);
                const q = deck.questions.find(item => item.id === qId);

                document.getElementById('q-type').value = q.type || 'flashcard';
                toggleQuestionTypeFields();
                document.getElementById('q-text').value = q.text || '';
                document.getElementById('q-image').value = q.image || '';
                document.getElementById('q-url').value = q.url || '';
                document.getElementById('q-explanation').value = q.explanation || '';
                document.getElementById('q-flashcard-answer').value = q.flashcardAnswer || '';
                document.getElementById('q-correct-text').value = q.correctText || '';
                renderChoiceInputs(q.choices && q.choices.length > 0 ? q.choices : ['', ''], q.correctIndex || 0);
            } else {
                titleEl.textContent = '問題の追加';
                document.getElementById('q-edit-id').value = '';
                document.getElementById('q-type').value = 'flashcard';
                toggleQuestionTypeFields();
                document.getElementById('q-text').value = '';
                document.getElementById('q-image').value = '';
                document.getElementById('q-url').value = '';
                document.getElementById('q-explanation').value = '';
                document.getElementById('q-flashcard-answer').value = '';
                document.getElementById('q-correct-text').value = '';
                renderChoiceInputs(['', '', '', ''], 0);
            }
            document.getElementById('modal-question').classList.add('active');
        }

        function saveQuestion() {
            const deckId = document.getElementById('q-deck-id').value;
            const editId = document.getElementById('q-edit-id').value;
            const type = document.getElementById('q-type').value;
            const text = document.getElementById('q-text').value.trim();
            const image = document.getElementById('q-image').value.trim();
            const url = document.getElementById('q-url').value.trim();
            const explanation = document.getElementById('q-explanation').value.trim();

            if (!text) return alert('問題文を入力してください。');

            let choices = [];
            let correctIndex = 0;
            let correctText = '';
            let flashcardAnswer = '';

            if (type === 'flashcard') {
                flashcardAnswer = document.getElementById('q-flashcard-answer').value.trim();
                if (!flashcardAnswer) return alert('解答を入力してください。');
            } else if (type === 'choice') {
                const choiceInputs = document.querySelectorAll('.choice-input');
                choiceInputs.forEach(i => {
                    if (i.value.trim()) choices.push(i.value.trim());
                });
                if (choices.length < 2) return alert('選択肢を2つ以上入力してください。');
                const radio = document.querySelector('input[name="q-correct"]:checked');
                correctIndex = radio ? parseInt(radio.value) : 0;
            } else if (type === 'input') {
                correctText = document.getElementById('q-correct-text').value.trim();
                if (!correctText) return alert('正解文字列を入力してください。');
            }

            const deck = decks.find(d => d.id === deckId);
            if (!deck) return;

            const newQ = {
                id: editId || ('q_' + Date.now()),
                type, text, image, url, explanation, flashcardAnswer, choices, correctIndex, correctText,
                nextReviewDate: null,
                totalAttempts: 0,
                correctCount: 0,
                memoText: '',
                memoImagePath: ''
            };

            if (editId) {
                const idx = deck.questions.findIndex(q => q.id === editId);
                if (idx !== -1) deck.questions[idx] = { ...deck.questions[idx], ...newQ };
            } else {
                deck.questions.push(newQ);
            }

            saveData();
            closeModal('modal-question');
            renderHome();
        }

        function deleteQuestion(deckId, qId) {
            if (confirm('この問題を削除しますか？')) {
                const deck = decks.find(d => d.id === deckId);
                if (deck) {
                    deck.questions = deck.questions.filter(q => q.id !== qId);
                    saveData();
                    renderHome();
                }
            }
        }

        // --- 手書きキャンバス実装 (要件定義書 2.3 準拠) ---
        function initCanvas() {
            memoCanvas = document.getElementById('memo-canvas');
            if (!memoCanvas) return;
            memoCtx = memoCanvas.getContext('2d');
            memoCtx.lineWidth = 2.5;
            memoCtx.lineCap = 'round';

            // マウスイベント
            memoCanvas.addEventListener('mousedown', startDrawing);
            memoCanvas.addEventListener('mousemove', draw);
            memoCanvas.addEventListener('mouseup', stopDrawing);
            memoCanvas.addEventListener('mouseout', stopDrawing);

            // タッチイベント (スマホ・タブレット)
            memoCanvas.addEventListener('touchstart', (e) => {
                const touch = e.touches[0];
                const rect = memoCanvas.getBoundingClientRect();
                startDrawing({ clientX: touch.clientX, clientY: touch.clientY });
            });
            memoCanvas.addEventListener('touchmove', (e) => {
                const touch = e.touches[0];
                draw({ clientX: touch.clientX, clientY: touch.clientY });
                e.preventDefault();
            });
            memoCanvas.addEventListener('touchend', stopDrawing);
        }

        function startDrawing(e) {
            isDrawing = true;
            const rect = memoCanvas.getBoundingClientRect();
            lastX = (e.clientX - rect.left) * (memoCanvas.width / rect.width);
            lastY = (e.clientY - rect.top) * (memoCanvas.height / rect.height);
        }

        function draw(e) {
            if (!isDrawing) return;
            const rect = memoCanvas.getBoundingClientRect();
            const currentX = (e.clientX - rect.left) * (memoCanvas.width / rect.width);
            const currentY = (e.clientY - rect.top) * (memoCanvas.height / rect.height);

            memoCtx.strokeStyle = penColor;
            memoCtx.beginPath();
            memoCtx.moveTo(lastX, lastY);
            memoCtx.lineTo(currentX, currentY);
            memoCtx.stroke();

            lastX = currentX;
            lastY = currentY;
        }

        function stopDrawing() { isDrawing = false; }
        function setPenColor(color) { penColor = color; }
        function clearMemoCanvas() { memoCtx.clearRect(0, 0, memoCanvas.width, memoCanvas.height); }

        function toggleMemoSection() {
            const sec = document.getElementById('memo-section');
            const txt = document.getElementById('memo-toggle-text');
            sec.classList.toggle('hidden');
            txt.textContent = sec.classList.contains('hidden') ? '思考プロセス・計算メモを開く' : '思考プロセス・計算メモを閉じる';
        }

        function switchMemoTab(tab) {
            if (tab === 'text') {
                document.getElementById('memo-text-container').classList.remove('hidden');
                document.getElementById('memo-draw-container').classList.add('hidden');
                document.getElementById('tab-memo-text').className = 'text-xs font-bold px-3 py-1 rounded bg-indigo-600 text-white';
                document.getElementById('tab-memo-draw').className = 'text-xs font-bold px-3 py-1 rounded bg-slate-200 text-slate-700';
            } else {
                document.getElementById('memo-text-container').classList.add('hidden');
                document.getElementById('memo-draw-container').classList.remove('hidden');
                document.getElementById('tab-memo-text').className = 'text-xs font-bold px-3 py-1 rounded bg-slate-200 text-slate-700';
                document.getElementById('tab-memo-draw').className = 'text-xs font-bold px-3 py-1 rounded bg-indigo-600 text-white';
            }
        }

        function togglePreviousMemoModal() {
            const q = currentTestState.questions[currentTestState.currentIndex];
            if (!q) return;

            document.getElementById('prev-memo-text-display').textContent = q.memoText || '前回のテキストメモはありません';
            const imgEl = document.getElementById('prev-memo-img-display');
            const noneEl = document.getElementById('prev-memo-img-none');

            if (q.memoImagePath) {
                imgEl.src = q.memoImagePath;
                imgEl.classList.remove('hidden');
                noneEl.classList.add('hidden');
            } else {
                imgEl.classList.add('hidden');
                noneEl.classList.remove('hidden');
            }

            document.getElementById('modal-prev-memo').classList.add('active');
        }

        // --- 学習・テスト実行ロジック ---
        function startTest(deckId, isReview = false, specificQId = null) {
            const deck = decks.find(d => d.id === deckId);
            if (!deck || deck.questions.length === 0) return;

            currentTestState.deck = deck;
            if (specificQId) {
                currentTestState.questions = JSON.parse(JSON.stringify(deck.questions.filter(q => q.id === specificQId)));
            } else {
                const order = document.getElementById('setting-order').value;
                let qs = JSON.parse(JSON.stringify(deck.questions));
                if (order === 'random') qs.sort(() => Math.random() - 0.5);
                currentTestState.questions = qs;
            }

            currentTestState.currentIndex = 0;
            currentTestState.correctCount = 0;
            currentTestState.isReviewMode = isReview;

            document.getElementById('test-deck-category').textContent = deck.name;
            ['home', 'review', 'achievements', 'settings', 'result'].forEach(v => document.getElementById(`view-${v}`).classList.add('hidden'));
            document.getElementById('view-test').classList.remove('hidden');
            renderTestQuestion();
        }

        function startReviewSession(deckId, qId) {
            startTest(deckId, true, qId);
        }

        function renderTestQuestion() {
            const q = currentTestState.questions[currentTestState.currentIndex];
            document.getElementById('current-q-num').textContent = currentTestState.currentIndex + 1;
            document.getElementById('total-q-num').textContent = currentTestState.questions.length;
            document.getElementById('test-deck-title').textContent = q.text;
            document.getElementById('test-q-text').textContent = q.text;
            document.getElementById('test-q-type-badge').textContent = getQTypeLabel(q.type);

            const imgContainer = document.getElementById('test-img-container');
            const imgEl = document.getElementById('test-img');
            if (q.image) {
                imgEl.src = q.image;
                imgContainer.classList.remove('hidden');
            } else {
                imgContainer.classList.add('hidden');
            }

            // メモとキャンバスクリア
            document.getElementById('test-memo-text').value = '';
            clearMemoCanvas();

            // 前回のメモの有無でボタン強調
            const btnPrevMemo = document.getElementById('btn-view-prev-memo');
            if (q.memoText || q.memoImagePath) {
                btnPrevMemo.classList.remove('opacity-50');
            } else {
                btnPrevMemo.classList.add('opacity-50');
            }

            // フィードバック非表示
            document.getElementById('test-feedback').classList.add('hidden');
            document.getElementById('test-unanswered-area').classList.remove('hidden');
            
            const flashcardCtrls = document.getElementById('test-flashcard-controls');
            const choicesContainer = document.getElementById('test-choices');
            const inputContainer = document.getElementById('test-input-container');

            flashcardCtrls.classList.add('hidden');
            choicesContainer.classList.add('hidden');
            inputContainer.classList.add('hidden');

            if (q.type === 'flashcard') {
                flashcardCtrls.classList.remove('hidden');
            } else if (q.type === 'choice') {
                choicesContainer.classList.remove('hidden');
                choicesContainer.innerHTML = '';
                q.choices.forEach((choice, index) => {
                    const btn = document.createElement('button');
                    btn.className = 'choice-btn w-full text-left p-3.5 bg-white border-2 border-slate-200 rounded-xl hover:border-indigo-500 hover:bg-indigo-50/50 font-medium transition text-sm text-slate-800 flex justify-between items-center';
                    btn.textContent = choice;
                    btn.onclick = () => submitChoiceAnswer(index, btn);
                    choicesContainer.appendChild(btn);
                });
            } else if (q.type === 'input') {
                inputContainer.classList.remove('hidden');
                document.getElementById('test-text-answer').value = '';
            }
        }

        function revealAnswer() {
            const q = currentTestState.questions[currentTestState.currentIndex];
            showFeedback(true, q.flashcardAnswer);
        }

        function submitChoiceAnswer(selectedIndex, clickedBtn) {
            const q = currentTestState.questions[currentTestState.currentIndex];
            const isCorrect = selectedIndex === q.correctIndex;
            
            document.querySelectorAll('.choice-btn').forEach(b => b.disabled = true);
            if (isCorrect) {
                clickedBtn.classList.add('border-emerald-500', 'bg-emerald-50', 'text-emerald-900');
            } else {
                clickedBtn.classList.add('border-red-500', 'bg-red-50', 'text-red-900');
            }

            showFeedback(isCorrect, `正解: ${q.choices[q.correctIndex]}`);
        }

        function submitTextAnswer() {
            const q = currentTestState.questions[currentTestState.currentIndex];
            const userVal = document.getElementById('test-text-answer').value.trim();
            const isCorrect = userVal === q.correctText;
            showFeedback(isCorrect, `正解: ${q.correctText}`);
        }

        function showFeedback(isCorrect, answerText) {
            const q = currentTestState.questions[currentTestState.currentIndex];
            document.getElementById('test-unanswered-area').classList.add('hidden');

            const feedbackEl = document.getElementById('test-feedback');
            const badgeEl = document.getElementById('feedback-badge');
            const expEl = document.getElementById('feedback-explanation');
            const urlContainer = document.getElementById('feedback-url-container');
            const urlEl = document.getElementById('feedback-url');

            feedbackEl.classList.remove('hidden', 'bg-emerald-50/50', 'bg-red-50/50', 'border-emerald-200', 'border-red-200');

            if (isCorrect) {
                feedbackEl.classList.add('bg-emerald-50/50', 'border-emerald-200');
                badgeEl.className = 'font-bold text-base text-emerald-700 flex items-center gap-1.5';
                badgeEl.innerHTML = '<i class="fa-solid fa-circle-check text-xl"></i> 正解！';
                currentTestState.correctCount++;
            } else {
                feedbackEl.classList.add('bg-red-50/50', 'border-red-200');
                badgeEl.className = 'font-bold text-base text-red-700 flex items-center gap-1.5';
                badgeEl.innerHTML = '<i class="fa-solid fa-circle-xmark text-xl"></i> 不正解...';
            }

            expEl.textContent = `${answerText}\n${q.explanation || ''}`;

            if (q.url) {
                urlEl.href = q.url;
                urlContainer.classList.remove('hidden');
            } else {
                urlContainer.classList.add('hidden');
            }
        }

        // 間隔反復（スペースド・レピティション）評価 & ドーパミン演出トリガー
        function rateQuestion(ratingLevel) {
            const q = currentTestState.questions[currentTestState.currentIndex];
            
            // 復習予定日の計算 (要件定義書 2.4 準拠)
            const today = new Date();
            let daysToAdd = 1;
            if (ratingLevel === 2) daysToAdd = 3;
            if (ratingLevel === 3) daysToAdd = 7;
            if (ratingLevel === 4) daysToAdd = 14;

            today.setDate(today.getDate() + daysToAdd);
            const nextDateStr = today.toISOString().split('T')[0];

            // 思考メモの保存 (要件定義書 2.3 & 2.7 準拠: ローカル保存/パス記録)
            const textMemo = document.getElementById('test-memo-text').value.trim();
            let imageMemoData = '';
            
            // キャンバスに描画があるか判定
            const isCanvasBlank = !memoCtx.getImageData(0, 0, memoCanvas.width, memoCanvas.height).data.some(channel => channel !== 0);
            if (!isCanvasBlank) {
                imageMemoData = memoCanvas.toDataURL('image/png');
            }

            // 元のデータストアを更新
            const targetDeck = decks.find(d => d.id === currentTestState.deck.id);
            if (targetDeck) {
                const targetQ = targetDeck.questions.find(item => item.id === q.id);
                if (targetQ) {
                    targetQ.nextReviewDate = nextDateStr;
                    targetQ.totalAttempts = (targetQ.totalAttempts || 0) + 1;
                    if (ratingLevel >= 3) targetQ.correctCount = (targetQ.correctCount || 0) + 1;
                    if (textMemo) targetQ.memoText = textMemo;
                    if (imageMemoData) targetQ.memoImagePath = imageMemoData;
                }
            }
            saveData();

            // 最高評価「簡単」選択時の爽快ドパガキ演出
            if (ratingLevel === 4) {
                triggerDopamineEffect();
            } else if (ratingLevel >= 2) {
                playDopamineSound('correct');
            }

            // 次の問題へ進む
            currentTestState.currentIndex++;
            if (currentTestState.currentIndex < currentTestState.questions.length) {
                renderTestQuestion();
            } else {
                finishTestSession();
            }
        }

        function finishTestSession() {
            ['home', 'review', 'achievements', 'settings', 'test'].forEach(v => document.getElementById(`view-${v}`).classList.add('hidden'));
            document.getElementById('view-result').classList.remove('hidden');

            const totalQ = currentTestState.questions.length;
            const accuracy = Math.round((currentTestState.correctCount / totalQ) * 100);

            document.getElementById('result-score').textContent = accuracy;
            const msgEl = document.getElementById('result-message');

            if (accuracy === 100) {
                msgEl.textContent = '完璧です！全問正解を達成しました！';
                unlockAchievement('perfect_score');
                triggerDopamineEffect();
            } else if (accuracy >= 80) {
                msgEl.textContent = '素晴らしい成績です！記憶がしっかり定着しています。';
            } else {
                msgEl.textContent = 'お疲れ様でした。間違えた問題は復習タスクで確認しましょう！';
            }

            unlockAchievement('first_quiz');
        }

        // --- ドーパミン促進演出 & サウンド (要件定義書 2.9 準拠) ---
        function triggerDopamineEffect() {
            const effectLevel = document.getElementById('setting-effect').value;
            if (effectLevel === 'quiet') return;

            // 紙吹雪 (Confetti) エフェクト
            if (typeof confetti === 'function') {
                confetti({
                    particleCount: 120,
                    spread: 80,
                    origin: { y: 0.6 }
                });
            }

            playDopamineSound('fanfare');

            if (effectLevel === 'max') {
                const body = document.getElementById('app-body');
                body.classList.add('dopamine-mode');
                setTimeout(() => body.classList.remove('dopamine-mode'), 1200);
            }
        }

        function playDopamineSound(type) {
            try {
                if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                if (audioCtx.state === 'suspended') audioCtx.resume();

                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.connect(gain);
                gain.connect(audioCtx.destination);

                if (type === 'correct') {
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(523.25, audioCtx.currentTime); // C5
                    osc.frequency.exponentialRampToValueAtTime(659.25, audioCtx.currentTime + 0.15); // E5
                    gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
                    osc.start();
                    osc.stop(audioCtx.currentTime + 0.3);
                } else if (type === 'fanfare' || type === 'achievement') {
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(523.25, audioCtx.currentTime);
                    osc.frequency.setValueAtTime(659.25, audioCtx.currentTime + 0.1);
                    osc.frequency.setValueAtTime(783.99, audioCtx.currentTime + 0.2);
                    osc.frequency.setValueAtTime(1046.50, audioCtx.currentTime + 0.3);
                    gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.6);
                    osc.start();
                    osc.stop(audioCtx.currentTime + 0.6);
                }
            } catch(e) {}
        }

        // --- エクスポート & インポート (要件定義書 2.6, 2.7 準拠) ---
        function exportData() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify({ decks, achievements, stats }));
            const downloadAnchor = document.createElement('a');
            downloadAnchor.setAttribute("href", dataStr);
            downloadAnchor.setAttribute("download", `MasterQuiz_Backup_${new Date().toISOString().split('T')[0]}.json`);
            document.body.appendChild(downloadAnchor);
            downloadAnchor.click();
            downloadAnchor.remove();
        }

        function importData(e) {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = (event) => {
                try {
                    const parsed = JSON.parse(event.target.result);
                    if (parsed.decks && Array.isArray(parsed.decks)) {
                        decks = parsed.decks;
                        if (parsed.achievements) achievements = parsed.achievements;
                        if (parsed.stats) stats = parsed.stats;
                        saveData();
                        renderHome();
                        alert('データを正常にインポートしました！');
                    } else {
                        alert('無効なファイルフォーマットです。');
                    }
                } catch(err) {
                    alert('ファイルの読み込みに失敗しました。');
                }
            };
            reader.readAsText(file);
        }

        function resetAllData() {
            if (confirm('本当にすべてのデータを削除して初期化しますか？この操作は取り消せません。')) {
                localStorage.removeItem('masterquiz_v2_decks');
                localStorage.removeItem('masterquiz_v2_achievements');
                localStorage.removeItem('masterquiz_v2_stats');
                location.reload();
            }
        }

        function escapeHtml(str) {
            if (!str) return '';
            return str.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;').replace(/'/g, '&#039;');
        }
    </script>
</body>
</html>
