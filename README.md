<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>离职申请表 - PDF生成器</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Ma+Shan+Zheng&family=Noto+Serif+SC:wght@400;600;700&display=swap');
        
        body {
            font-family: 'Noto Serif SC', 'SimSun', '宋体', serif;
            background: #f3f4f6;
        }
        
        .form-input {
            transition: all 0.2s;
        }
        .form-input:focus {
            outline: none;
            border-color: #3b82f6;
            box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
        }
        
        .preview-page {
            width: 210mm;
            min-height: 297mm;
            background: white;
            margin: 0 auto;
            padding: 20mm 15mm 15mm 15mm;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            position: relative;
            font-family: 'Noto Serif SC', 'SimSun', '宋体', serif;
            font-size: 10.5pt;
            line-height: 1.6;
            color: #000;
        }
        
        .preview-table {
            width: 100%;
            border-collapse: collapse;
            table-layout: fixed;
        }
        
        .preview-table td, .preview-table th {
            border: 1px solid #000;
            padding: 8px 10px;
            vertical-align: middle;
        }
        
        .preview-table .label-cell {
            width: 15%;
            text-align: center;
            font-weight: normal;
            background: #fff;
        }
        
        .preview-table .value-cell {
            width: 18%;
            text-align: center;
        }
        
        .statement-text {
            text-indent: 2em;
            text-align: justify;
            line-height: 2.2;
            padding: 15px;
        }
        
        .signature-line {
            border-bottom: 1px solid #000;
            display: inline-block;
            min-width: 120px;
            text-align: center;
            padding: 0 10px;
            position: relative;
            height: 36px;
            vertical-align: bottom;
        }
        
        .handwriting-text {
            font-family: 'Ma Shan Zheng', cursive;
            font-size: 26pt;
            color: #000;
            display: inline-block;
            transform: rotate(-2deg);
            line-height: 1;
        }
        
        .handwriting-img {
            max-height: 40px;
            max-width: 140px;
            display: inline-block;
            transform: rotate(-2deg);
        }
        
        @media print {
            body * { visibility: hidden; }
            #preview-area, #preview-area * { visibility: visible; }
            #preview-area {
                position: absolute;
                left: 0; top: 0;
                margin: 0;
                box-shadow: none;
            }
        }
        
        .btn-primary {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(102, 126, 234, 0.3);
        }
        
        .input-group {
            background: white;
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }
        
        .signature-pad {
            border: 2px dashed #cbd5e1;
            border-radius: 8px;
            cursor: crosshair;
            background: #fafafa;
            touch-action: none;
        }
        .signature-pad:hover {
            border-color: #94a3b8;
        }
        
        .modal-overlay {
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.5);
            z-index: 100;
            display: none;
            align-items: center;
            justify-content: center;
            backdrop-filter: blur(4px);
        }
        .modal-overlay.active {
            display: flex;
        }
        .modal-content {
            background: white;
            border-radius: 16px;
            padding: 32px;
            max-width: 480px;
            width: 90%;
            text-align: center;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
        }
        
        .toast {
            position: fixed;
            top: 80px;
            left: 50%;
            transform: translateX(-50%) translateY(-20px);
            background: #10b981;
            color: white;
            padding: 12px 24px;
            border-radius: 8px;
            font-weight: 500;
            box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1);
            opacity: 0;
            transition: all 0.3s;
            z-index: 200;
            pointer-events: none;
        }
        .toast.show {
            opacity: 1;
            transform: translateX(-50%) translateY(0);
        }
        
        .data-textarea {
            font-family: monospace;
            font-size: 11px;
            line-height: 1.4;
            background: #f8fafc;
            border: 1px solid #e2e8f0;
            border-radius: 8px;
            padding: 12px;
            width: 100%;
            min-height: 80px;
            resize: vertical;
            word-break: break-all;
        }
        
        .tab-btn {
            padding: 8px 16px;
            border-radius: 8px;
            font-size: 14px;
            font-weight: 500;
            transition: all 0.2s;
            cursor: pointer;
            border: 1px solid #e5e7eb;
            background: white;
            color: #374151;
        }
        .tab-btn.active {
            background: #3b82f6;
            color: white;
            border-color: #3b82f6;
        }
        
        /* 扫码引导区域 */
        .scan-guide {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border-radius: 16px;
            padding: 24px;
            color: white;
            text-align: center;
            margin-bottom: 20px;
        }
        
        .scan-guide h3 {
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 12px;
        }
        
        .scan-guide .qr-box {
            background: white;
            padding: 16px;
            border-radius: 12px;
            display: inline-block;
            margin: 12px 0;
        }
        
        .scan-guide p {
            font-size: 14px;
            opacity: 0.9;
            margin-top: 8px;
        }
        
        .env-badge {
            display: inline-block;
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: 500;
            margin-bottom: 12px;
        }
        
        .env-badge.online {
            background: #dcfce7;
            color: #166534;
        }
        
        .env-badge.local {
            background: #fef3c7;
            color: #92400e;
        }
        
        @media (max-width: 768px) {
            .preview-page {
                width: 100%;
                padding: 10mm 8mm;
                font-size: 9pt;
            }
            .handwriting-text {
                font-size: 20pt;
            }
            .scan-guide {
                padding: 16px;
            }
        }
    </style>
</head>
<body class="min-h-screen">

    <div id="toast" class="toast"></div>

    <!-- 头部 -->
    <div class="bg-white shadow-sm border-b border-gray-200 sticky top-0 z-50">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
            <div class="flex items-center gap-3">
                <div class="w-8 h-8 bg-gradient-to-br from-blue-500 to-purple-600 rounded-lg flex items-center justify-center text-white font-bold text-sm">PDF</div>
                <h1 class="text-xl font-bold text-gray-800">离职申请表生成器</h1>
            </div>
            <div class="flex gap-3" id="desktop-actions">
                <button onclick="showReceiveModal()" class="px-4 py-2 text-sm font-medium text-green-700 bg-green-50 border border-green-200 rounded-lg hover:bg-green-100 transition-colors">
                    📲 接收手机数据
                </button>
                <button onclick="window.print()" class="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors">
                    🖨️ 浏览器打印
                </button>
                <button onclick="generatePDF()" class="px-4 py-2 text-sm font-medium text-white bg-blue-600 rounded-lg hover:bg-blue-700 transition-colors shadow-lg shadow-blue-500/30">
                    📄 生成 PDF 下载
                </button>
            </div>
        </div>
    </div>

    <!-- 扫码引导弹窗（部署后自动显示） -->
    <div id="scan-guide-modal" class="modal-overlay active">
        <div class="modal-content" style="max-width: 420px;">
            <div class="scan-guide" style="margin: -32px -32px 20px -32px; border-radius: 16px 16px 0 0;">
                <div class="env-badge online" id="env-badge">🌐 在线部署模式</div>
                <h3>📱 扫码填写离职申请表</h3>
                <div class="qr-box" id="guide-qrcode"></div>
                <p>请让员工用手机扫描上方二维码填写</p>
                <p style="font-size: 12px; margin-top: 4px;">填写完成后会自动传回此电脑</p>
            </div>
            
            <div class="text-left mb-4">
                <p class="text-sm font-medium text-gray-700 mb-2">当前访问地址：</p>
                <div class="flex gap-2">
                    <input type="text" id="current-url" readonly class="flex-1 px-3 py-2 bg-gray-50 border border-gray-200 rounded-lg text-xs font-mono text-gray-600">
                    <button onclick="copyCurrentURL()" class="px-3 py-2 bg-blue-600 text-white rounded-lg text-sm font-medium hover:bg-blue-700 transition-colors">
                        复制
                    </button>
                </div>
            </div>
            
            <div class="bg-blue-50 border border-blue-100 rounded-lg p-3 mb-4 text-left">
                <p class="text-xs text-blue-800 font-medium">💡 使用流程</p>
                <p class="text-xs text-blue-700 mt-1">1. 员工扫码填写 → 2. 点击"传回电脑" → 3. 您点击"接收手机数据"</p>
            </div>
            
            <button onclick="hideScanGuide()" class="w-full py-2 bg-gray-100 hover:bg-gray-200 rounded-lg text-sm font-medium text-gray-700 transition-colors">
                我知道了，开始填表
            </button>
        </div>
    </div>

    <!-- 手机传输弹窗 -->
    <div id="transfer-modal" class="modal-overlay" onclick="if(event.target===this)hideTransferModal()">
        <div class="modal-content">
            <h3 class="text-lg font-bold text-gray-800 mb-2">将数据传回电脑</h3>
            <p class="text-sm text-gray-500 mb-4">选择以下任一方式，将填写好的数据发送到电脑</p>
            
            <div class="flex gap-2 justify-center mb-4">
                <button onclick="switchTransferTab('qrcode')" id="tab-qrcode" class="tab-btn active">📷 二维码</button>
                <button onclick="switchTransferTab('text')" id="tab-text" class="tab-btn">📋 复制密文</button>
                <button onclick="switchTransferTab('file')" id="tab-file" class="tab-btn">📁 导出文件</button>
            </div>
            
            <div id="panel-qrcode">
                <div id="transfer-qrcode-container" class="flex justify-center mb-3"></div>
                <p class="text-xs text-gray-500 mb-2">用电脑浏览器扫描上方二维码（需同网络）</p>
                <p class="text-xs text-orange-600 bg-orange-50 rounded p-2 mb-3">⚠️ 如果扫描后无法打开，请换用"复制密文"方式</p>
            </div>
            
            <div id="panel-text" class="hidden">
                <div class="text-left mb-3">
                    <p class="text-sm text-gray-700 mb-2 font-medium">步骤：</p>
                    <p class="text-xs text-gray-500 mb-3">1. 点击下方"复制密文" → 2. 通过微信/钉钉发给电脑 → 3. 电脑端点击"接收手机数据"粘贴</p>
                </div>
                <textarea id="data-cipher" readonly class="data-textarea mb-3"></textarea>
                <button onclick="copyCipher()" class="w-full py-2 bg-blue-600 text-white rounded-lg text-sm font-medium hover:bg-blue-700 transition-colors mb-2">
                    📋 复制密文
                </button>
            </div>
            
            <div id="panel-file" class="hidden">
                <p class="text-sm text-gray-600 mb-3">下载包含数据的HTML文件，发送到电脑后打开即可</p>
                <button onclick="exportDataFile()" class="w-full py-2 bg-purple-600 text-white rounded-lg text-sm font-medium hover:bg-purple-700 transition-colors">
                    📥 下载数据文件
                </button>
            </div>
            
            <button onclick="hideTransferModal()" class="w-full py-2 bg-gray-100 hover:bg-gray-200 rounded-lg text-sm font-medium text-gray-700 transition-colors mt-3">
                关闭
            </button>
        </div>
    </div>

    <!-- 电脑接收数据弹窗 -->
    <div id="receive-modal" class="modal-overlay" onclick="if(event.target===this)hideReceiveModal()">
        <div class="modal-content">
            <h3 class="text-lg font-bold text-gray-800 mb-2">📲 接收手机数据</h3>
            <p class="text-sm text-gray-500 mb-4">将手机端复制的密文粘贴到下方，或选择导入数据文件</p>
            
            <div class="flex gap-2 justify-center mb-4">
                <button onclick="switchReceiveTab('paste')" id="tab-receive-paste" class="tab-btn active">📋 粘贴密文</button>
                <button onclick="switchReceiveTab('file')" id="tab-receive-file" class="tab-btn">📁 导入文件</button>
            </div>
            
            <div id="panel-receive-paste">
                <textarea id="receive-cipher" class="data-textarea mb-3" placeholder="将手机端复制的密文粘贴到这里..."></textarea>
                <button onclick="receiveFromCipher()" class="w-full py-2 bg-green-600 text-white rounded-lg text-sm font-medium hover:bg-green-700 transition-colors">
                    ✅ 解析并填充数据
                </button>
            </div>
            
            <div id="panel-receive-file" class="hidden">
                <input type="file" id="receive-file-input" accept=".html,.json,.txt" onchange="receiveFromFile(this)" class="hidden">
                <button onclick="document.getElementById('receive-file-input').click()" class="w-full py-2 bg-purple-600 text-white rounded-lg text-sm font-medium hover:bg-purple-700 transition-colors">
                    📁 选择数据文件
                </button>
                <p class="text-xs text-gray-500 mt-2">支持 .html / .json / .txt 格式</p>
            </div>
            
            <button onclick="hideReceiveModal()" class="w-full py-2 bg-gray-100 hover:bg-gray-200 rounded-lg text-sm font-medium text-gray-700 transition-colors mt-3">
                关闭
            </button>
        </div>
    </div>

    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8" id="main-layout">
            
            <!-- 左侧：表单 -->
            <div class="space-y-6" id="form-section">
                
                <!-- 扫码引导卡片（电脑端显示） -->
                <div class="scan-guide hidden" id="scan-guide-card">
                    <div class="env-badge online">🌐 在线部署模式</div>
                    <h3>📱 让员工扫码填写</h3>
                    <div class="qr-box" id="card-qrcode"></div>
                    <p>手机扫描上方二维码即可填写</p>
                    <button onclick="showScanGuideModal()" class="mt-3 px-4 py-2 bg-white text-purple-700 rounded-lg text-sm font-medium hover:bg-gray-100 transition-colors">
                        放大二维码
                    </button>
                </div>

                <div class="input-group">
                    <h2 class="text-lg font-bold text-gray-800 mb-4 flex items-center gap-2">
                        <span class="w-1 h-6 bg-blue-500 rounded-full"></span>
                        基本信息
                    </h2>
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">姓名 <span class="text-red-500">*</span></label>
                            <input type="text" id="input-name" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg" placeholder="请输入姓名">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">出生年月 <span class="text-red-500">*</span></label>
                            <input type="date" id="input-birth" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">岗位 <span class="text-red-500">*</span></label>
                            <input type="text" id="input-position" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg" placeholder="例如：BD">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">最后工作日 <span class="text-red-500">*</span></label>
                            <input type="date" id="input-lastday" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg">
                        </div>
                    </div>
                </div>

                <div class="input-group">
                    <h2 class="text-lg font-bold text-gray-800 mb-4 flex items-center gap-2">
                        <span class="w-1 h-6 bg-purple-500 rounded-full"></span>
                        联系方式
                    </h2>
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">个人邮箱</label>
                            <input type="text" id="input-email" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg" placeholder="选填">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">个人联系电话 <span class="text-red-500">*</span></label>
                            <input type="text" id="input-phone" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg" placeholder="请输入手机号">
                        </div>
                    </div>
                </div>

                <div class="input-group">
                    <h2 class="text-lg font-bold text-gray-800 mb-4 flex items-center gap-2">
                        <span class="w-1 h-6 bg-green-500 rounded-full"></span>
                        身份信息
                    </h2>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">身份证号码 <span class="text-red-500">*</span></label>
                        <input type="text" id="input-idcard" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg" placeholder="请输入18位身份证号码">
                    </div>
                </div>

                <div class="input-group">
                    <h2 class="text-lg font-bold text-gray-800 mb-4 flex items-center gap-2">
                        <span class="w-1 h-6 bg-red-500 rounded-full"></span>
                        签署信息
                    </h2>
                    <div class="mb-4">
                        <div class="flex gap-2 mb-3">
                            <button onclick="setSignMode('text')" id="btn-text-mode" class="px-3 py-1.5 text-sm rounded-md bg-blue-100 text-blue-700 font-medium transition-colors">✏️ 手写体文字</button>
                            <button onclick="setSignMode('draw')" id="btn-draw-mode" class="px-3 py-1.5 text-sm rounded-md bg-gray-100 text-gray-600 hover:bg-gray-200 transition-colors">✍️ 手写签名板</button>
                        </div>
                        <div id="text-sign-panel">
                            <label class="block text-sm font-medium text-gray-700 mb-1">员工签名 <span class="text-red-500">*</span></label>
                            <input type="text" id="input-signature" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg" placeholder="请输入签名，将显示为手写体">
                        </div>
                        <div id="draw-sign-panel" class="hidden">
                            <div class="flex justify-between items-center mb-1">
                                <label class="block text-sm font-medium text-gray-700">在下方区域手写签名</label>
                                <div class="flex gap-2">
                                    <button onclick="clearSignature()" class="text-xs px-2 py-1 text-red-600 bg-red-50 rounded hover:bg-red-100">清空</button>
                                    <button onclick="confirmSignature()" class="text-xs px-2 py-1 text-blue-600 bg-blue-50 rounded hover:bg-blue-100">确认签名</button>
                                </div>
                            </div>
                            <canvas id="signature-canvas" class="signature-pad w-full" width="400" height="120"></canvas>
                            <p class="text-xs text-gray-500 mt-1">支持鼠标和触摸屏手写</p>
                        </div>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">签署日期 <span class="text-red-500">*</span></label>
                        <input type="date" id="input-sign-date" class="form-input w-full px-3 py-2 border border-gray-300 rounded-lg">
                    </div>
                </div>

                <button onclick="generatePDF()" class="btn-primary w-full py-3 rounded-xl text-white font-bold text-lg shadow-lg hidden" id="btn-desktop-generate">
                    📄 生成 PDF 文件
                </button>
                
                <button onclick="showTransferModal()" class="w-full py-3 rounded-xl text-white font-bold text-lg shadow-lg bg-gradient-to-r from-green-500 to-emerald-600 hover:from-green-600 hover:to-emerald-700 transition-all hidden" id="btn-mobile-transfer">
                    ✅ 填写完成，传回电脑生成PDF
                </button>
                
                <p class="text-xs text-gray-500 text-center" id="hint-text">
                    提示：也可以直接点击右上角"浏览器打印"，选择"另存为PDF"以获得最佳效果
                </p>
            </div>

            <!-- 右侧：预览 -->
            <div class="lg:sticky lg:top-24 lg:h-fit" id="preview-section">
                <div class="bg-gray-200 p-4 rounded-xl">
                    <div class="text-center text-sm text-gray-600 mb-2 font-medium">实时预览</div>
                    <div id="preview-area" class="preview-page">
                        <div style="font-size: 9pt; color: #333; margin-bottom: 8px;">RB2-L1-2025v1.0</div>
                        <div style="text-align: center; font-size: 18pt; font-weight: bold; letter-spacing: 8px; margin-bottom: 20px; margin-top: 10px;">离职申请表</div>

                        <table class="preview-table">
                            <tr>
                                <td class="label-cell">姓 名</td>
                                <td class="value-cell" id="preview-name"></td>
                                <td class="label-cell">出生年月</td>
                                <td class="value-cell" id="preview-birth"></td>
                                <td class="label-cell">岗 位</td>
                                <td class="value-cell" id="preview-position"></td>
                            </tr>
                            <tr>
                                <td class="label-cell">个人邮箱</td>
                                <td class="value-cell" id="preview-email" colspan="2"></td>
                                <td class="label-cell">个人联系电话</td>
                                <td class="value-cell" id="preview-phone" colspan="2"></td>
                            </tr>
                            <tr>
                                <td class="label-cell">身份证号码</td>
                                <td class="value-cell" id="preview-idcard" colspan="2"></td>
                                <td class="label-cell">最后工作日</td>
                                <td class="value-cell" id="preview-lastday" colspan="2"></td>
                            </tr>
                        </table>

                        <div style="border: 1px solid #000; border-top: none; padding: 0;">
                            <div class="statement-text">
                                本人因个人原因自愿解除与贵司签订的劳动合同。本人对与贵司劳动关系存续期间的其<br>
                                他一切事项（包括但不限于工资、福利、津贴、补助、绩效、加班工资、年休假工资、病假<br>
                                工资、报销款、工伤保险待遇、生育保险待遇、其他社保相关待遇）确认无任何争议。
                            </div>
                            <div style="padding: 0 15px 15px 15px; text-indent: 2em;">
                                以上，均为本人真实意思表示，本人确认以上信息。
                            </div>
                            <div style="padding: 10px 15px 20px 15px; display: flex; align-items: center; gap: 60px;">
                                <div style="display: flex; align-items: center; gap: 8px; white-space: nowrap;">
                                    <span>员工（签名）：</span>
                                    <span class="signature-line" id="preview-signature-box">
                                        <span class="handwriting-text" id="preview-signature"></span>
                                    </span>
                                </div>
                                <div style="display: flex; align-items: center; gap: 8px; white-space: nowrap;">
                                    <span>日期：</span>
                                    <span id="preview-sign-date"></span>
                                </div>
                            </div>
                        </div>

                        <div style="margin-top: 25px; line-height: 2.2;">
                            <div style="font-weight: bold; display: inline;">特别承诺：</div>
                            <span>本人在离职后，仍需履行保密义务，不得向任何单位或个人透露获知的公司或客户的</span><br>
                            <span>任何资料、信息等。公司已告知本人无需履行竞业限制义务，公司无需向本人支付竞业限制补偿</span><br>
                            <span>金。</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // ==================== 环境检测 ====================
        function detectEnvironment() {
            const protocol = window.location.protocol;
            const hostname = window.location.hostname;
            
            // 检测是否在线部署
            const isOnline = protocol === 'https:' && (hostname.includes('github.io') || hostname.includes('vercel.app') || hostname.includes('netlify.app') || hostname.includes('pages.dev'));
            
            // 检测是否本地文件
            const isLocalFile = protocol === 'file:';
            
            return { isOnline, isLocalFile, hostname };
        }
        
        // ==================== 扫码引导 ====================
        function initScanGuide() {
            const env = detectEnvironment();
            const modal = document.getElementById('scan-guide-modal');
            const card = document.getElementById('scan-guide-card');
            const badge = document.getElementById('env-badge');
            const currentUrlInput = document.getElementById('current-url');
            
            const currentUrl = window.location.href;
            currentUrlInput.value = currentUrl;
            
            // 生成二维码
            const guideQr = document.getElementById('guide-qrcode');
            const cardQr = document.getElementById('card-qrcode');
            
            guideQr.innerHTML = '';
            cardQr.innerHTML = '';
            
            new QRCode(guideQr, {
                text: currentUrl,
                width: 180,
                height: 180,
                colorDark: "#000000",
                colorLight: "#ffffff",
                correctLevel: QRCode.CorrectLevel.M
            });
            
            new QRCode(cardQr, {
                text: currentUrl,
                width: 120,
                height: 120,
                colorDark: "#000000",
                colorLight: "#ffffff",
                correctLevel: QRCode.CorrectLevel.M
            });
            
            if (env.isOnline) {
                // 在线部署：显示引导
                badge.textContent = '🌐 在线部署模式';
                badge.className = 'env-badge online';
                card.classList.remove('hidden');
                
                // 首次访问显示弹窗
                if (!localStorage.getItem('scanGuideSeen')) {
                    modal.classList.add('active');
                }
            } else if (env.isLocalFile) {
                // 本地文件
                badge.textContent = '💻 本地文件模式';
                badge.className = 'env-badge local';
                card.classList.add('hidden');
                modal.classList.remove('active');
            } else {
                // 本地服务器
                badge.textContent = '📡 本地服务器';
                badge.className = 'env-badge local';
                card.classList.remove('hidden');
                
                if (!localStorage.getItem('scanGuideSeen')) {
                    modal.classList.add('active');
                }
            }
        }
        
        function hideScanGuide() {
            document.getElementById('scan-guide-modal').classList.remove('active');
            localStorage.setItem('scanGuideSeen', 'true');
        }
        
        function showScanGuideModal() {
            document.getElementById('scan-guide-modal').classList.add('active');
        }
        
        function copyCurrentURL() {
            const url = document.getElementById('current-url');
            url.select();
            navigator.clipboard.writeText(url.value).then(() => {
                showToast('✅ 链接已复制');
            });
        }
        
        // ==================== 设备检测 ====================
        let isMobileDevice = false;
        
        function detectDevice() {
            const isMobile = window.innerWidth < 1024 || /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
            isMobileDevice = isMobile;
            
            if (isMobile) {
                document.getElementById('preview-section').style.display = 'none';
                document.getElementById('desktop-actions').style.display = 'none';
                document.getElementById('main-layout').classList.remove('grid-cols-1', 'lg:grid-cols-2');
                document.getElementById('main-layout').classList.add('grid-cols-1');
                document.getElementById('btn-desktop-generate').classList.add('hidden');
                document.getElementById('btn-mobile-transfer').classList.remove('hidden');
                document.getElementById('hint-text').textContent = '填写完成后，点击下方按钮将数据传回电脑';
                document.getElementById('scan-guide-card').classList.add('hidden');
            } else {
                document.getElementById('btn-desktop-generate').classList.remove('hidden');
                document.getElementById('btn-mobile-transfer').classList.add('hidden');
            }
        }
        
        // ==================== 数据编解码 ====================
        function getFormData() {
            return {
                name: document.getElementById('input-name').value,
                birth: document.getElementById('input-birth').value,
                position: document.getElementById('input-position').value,
                lastday: document.getElementById('input-lastday').value,
                email: document.getElementById('input-email').value,
                phone: document.getElementById('input-phone').value,
                idcard: document.getElementById('input-idcard').value,
                signature: document.getElementById('input-signature').value,
                signDate: document.getElementById('input-sign-date').value,
                signMode: signMode,
                handwritingImg: document.getElementById('preview-signature-img')?.src || null
            };
        }
        
        function encodeData(data) {
            const json = JSON.stringify(data);
            return btoa(unescape(encodeURIComponent(json)));
        }
        
        function decodeData(str) {
            try {
                const json = decodeURIComponent(escape(atob(str)));
                return JSON.parse(json);
            } catch (e) {
                return null;
            }
        }
        
        function fillForm(data) {
            if (!data) return false;
            
            document.getElementById('input-name').value = data.name || '';
            document.getElementById('input-birth').value = data.birth || '';
            document.getElementById('input-position').value = data.position || '';
            document.getElementById('input-lastday').value = data.lastday || '';
            document.getElementById('input-email').value = data.email || '';
            document.getElementById('input-phone').value = data.phone || '';
            document.getElementById('input-idcard').value = data.idcard || '';
            document.getElementById('input-signature').value = data.signature || '';
            document.getElementById('input-sign-date').value = data.signDate || '';
            
            if (data.signMode === 'draw' && data.handwritingImg) {
                setSignMode('draw');
                const box = document.getElementById('preview-signature-box');
                const text = document.getElementById('preview-signature');
                text.style.display = 'none';
                
                const oldImg = document.getElementById('preview-signature-img');
                if (oldImg) oldImg.remove();
                
                const img = document.createElement('img');
                img.id = 'preview-signature-img';
                img.src = data.handwritingImg;
                img.className = 'handwriting-img';
                img.style.maxHeight = '36px';
                box.appendChild(img);
            } else {
                setSignMode('text');
            }
            
            updatePreview();
            return true;
        }
        
        // ==================== 传输弹窗 ====================
        function showTransferModal() {
            if (!validateForm()) return;
            
            const modal = document.getElementById('transfer-modal');
            const qrcodeContainer = document.getElementById('transfer-qrcode-container');
            const cipherTextarea = document.getElementById('data-cipher');
            
            qrcodeContainer.innerHTML = '';
            const baseUrl = window.location.href.split('#')[0];
            const data = getFormData();
            const encoded = encodeData(data);
            const fullUrl = `${baseUrl}#data=${encoded}`;
            
            if (fullUrl.length < 2000) {
                new QRCode(qrcodeContainer, {
                    text: fullUrl,
                    width: 200,
                    height: 200,
                    colorDark: "#000000",
                    colorLight: "#ffffff",
                    correctLevel: QRCode.CorrectLevel.L
                });
            } else {
                qrcodeContainer.innerHTML = '<p class="text-sm text-orange-600">数据过长，请使用"复制密文"方式</p>';
            }
            
            cipherTextarea.value = encoded;
            modal.classList.add('active');
        }
        
        function hideTransferModal() {
            document.getElementById('transfer-modal').classList.remove('active');
        }
        
        function switchTransferTab(tab) {
            document.querySelectorAll('#transfer-modal .tab-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('tab-' + tab).classList.add('active');
            
            document.getElementById('panel-qrcode').classList.add('hidden');
            document.getElementById('panel-text').classList.add('hidden');
            document.getElementById('panel-file').classList.add('hidden');
            document.getElementById('panel-' + tab).classList.remove('hidden');
        }
        
        function copyCipher() {
            const textarea = document.getElementById('data-cipher');
            textarea.select();
            navigator.clipboard.writeText(textarea.value).then(() => {
                showToast('✅ 密文已复制，请发给电脑');
            });
        }
        
        function exportDataFile() {
            const data = getFormData();
            const encoded = encodeData(data);
            const htmlContent = `<!DOCTYPE html>
<html>
<head><meta charset="UTF-8"><title>离职申请表数据</title></head>
<body>
<script>
window.location.href = "${window.location.href.split('#')[0]}#data=${encoded}";
<<\/script>
<p>正在跳转...</p>
<p>如果没有自动跳转，请复制以下密文到电脑端"接收手机数据"中粘贴：</p>
<<textarea style="width:100%;height:100px">${encoded}</textarea>
</body>
</html>`;
            
            const blob = new Blob([htmlContent], { type: 'text/html' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = '离职申请表数据.html';
            a.click();
            URL.revokeObjectURL(url);
            showToast('📥 文件已下载，请发送到电脑打开');
        }
        
        // ==================== 接收弹窗 ====================
        function showReceiveModal() {
            document.getElementById('receive-modal').classList.add('active');
        }
        
        function hideReceiveModal() {
            document.getElementById('receive-modal').classList.remove('active');
        }
        
        function switchReceiveTab(tab) {
            document.querySelectorAll('#receive-modal .tab-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('tab-receive-' + tab).classList.add('active');
            
            document.getElementById('panel-receive-paste').classList.add('hidden');
            document.getElementById('panel-receive-file').classList.add('hidden');
            document.getElementById('panel-receive-' + tab).classList.remove('hidden');
        }
        
        function receiveFromCipher() {
            const cipher = document.getElementById('receive-cipher').value.trim();
            if (!cipher) {
                showToast('❌ 请输入密文');
                return;
            }
            
            const data = decodeData(cipher);
            if (data && fillForm(data)) {
                hideReceiveModal();
                showToast('📱 数据接收成功！请检查预览并生成PDF');
                setTimeout(() => {
                    document.getElementById('preview-section').scrollIntoView({ behavior: 'smooth' });
                }, 500);
            } else {
                showToast('❌ 密文解析失败，请检查是否复制完整');
            }
        }
        
        function receiveFromFile(input) {
            const file = input.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const content = e.target.result;
                const match = content.match(/#data=([A-Za-z0-9+/=]+)/);
                if (match) {
                    const data = decodeData(match[1]);
                    if (data && fillForm(data)) {
                        hideReceiveModal();
                        showToast('📱 文件导入成功！');
                        return;
                    }
                }
                
                try {
                    const data = decodeData(content.trim());
                    if (data && fillForm(data)) {
                        hideReceiveModal();
                        showToast('📱 文件导入成功！');
                        return;
                    }
                } catch (e) {}
                
                showToast('❌ 无法解析文件，请确认是导出的数据文件');
            };
            reader.readAsText(file);
        }
        
        function checkUrlHash() {
            if (location.hash.startsWith('#data=')) {
                const encoded = location.hash.slice(6);
                const data = decodeData(encoded);
                if (data && fillForm(data)) {
                    showToast('📱 已从链接接收数据！请检查预览并生成PDF');
                    history.replaceState(null, null, location.pathname + location.search);
                }
            }
        }
        
        // ==================== 表单验证 ====================
        function validateForm() {
            const required = ['input-name', 'input-birth', 'input-position', 'input-lastday', 'input-phone', 'input-idcard', 'input-sign-date'];
            let valid = true;
            required.forEach(id => {
                const el = document.getElementById(id);
                if (!el.value.trim()) {
                    el.style.borderColor = '#ef4444';
                    valid = false;
                } else {
                    el.style.borderColor = '#d1d5db';
                }
            });
            
            if (signMode === 'text' && !document.getElementById('input-signature').value.trim()) {
                document.getElementById('input-signature').style.borderColor = '#ef4444';
                valid = false;
            }
            
            if (!valid) showToast('❌ 请填写所有必填项');
            return valid;
        }
        
        function showToast(message) {
            const toast = document.getElementById('toast');
            toast.textContent = message;
            toast.classList.add('show');
            setTimeout(() => toast.classList.remove('show'), 3000);
        }
        
        // ==================== 签名板 ====================
        const canvas = document.getElementById('signature-canvas');
        const ctx = canvas.getContext('2d');
        let isDrawing = false;
        let hasDrawn = false;
        let signMode = 'text';

        ctx.strokeStyle = '#000';
        ctx.lineWidth = 2;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';

        function getPos(e) {
            const rect = canvas.getBoundingClientRect();
            const clientX = e.touches ? e.touches[0].clientX : e.clientX;
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            return {
                x: (clientX - rect.left) * (canvas.width / rect.width),
                y: (clientY - rect.top) * (canvas.height / rect.height)
            };
        }

        function startDraw(e) {
            e.preventDefault();
            isDrawing = true;
            hasDrawn = true;
            const pos = getPos(e);
            ctx.beginPath();
            ctx.moveTo(pos.x, pos.y);
        }

        function draw(e) {
            e.preventDefault();
            if (!isDrawing) return;
            const pos = getPos(e);
            ctx.lineTo(pos.x, pos.y);
            ctx.stroke();
        }

        function endDraw() {
            isDrawing = false;
        }

        canvas.addEventListener('mousedown', startDraw);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('mouseup', endDraw);
        canvas.addEventListener('mouseout', endDraw);
        canvas.addEventListener('touchstart', startDraw, { passive: false });
        canvas.addEventListener('touchmove', draw, { passive: false });
        canvas.addEventListener('touchend', endDraw);

        function clearSignature() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            hasDrawn = false;
            document.getElementById('preview-signature').style.display = '';
            const img = document.getElementById('preview-signature-img');
            if (img) img.remove();
        }

        function confirmSignature() {
            if (!hasDrawn) {
                showToast('请先手写签名');
                return;
            }
            const dataURL = canvas.toDataURL('image/png');
            const box = document.getElementById('preview-signature-box');
            const text = document.getElementById('preview-signature');
            text.style.display = 'none';
            
            const oldImg = document.getElementById('preview-signature-img');
            if (oldImg) oldImg.remove();
            
            const img = document.createElement('img');
            img.id = 'preview-signature-img';
            img.src = dataURL;
            img.className = 'handwriting-img';
            img.style.maxHeight = '36px';
            box.appendChild(img);
            showToast('签名已确认');
        }

        function setSignMode(mode) {
            signMode = mode;
            const textPanel = document.getElementById('text-sign-panel');
            const drawPanel = document.getElementById('draw-sign-panel');
            const btnText = document.getElementById('btn-text-mode');
            const btnDraw = document.getElementById('btn-draw-mode');
            
            if (mode === 'text') {
                textPanel.classList.remove('hidden');
                drawPanel.classList.add('hidden');
                btnText.className = 'px-3 py-1.5 text-sm rounded-md bg-blue-100 text-blue-700 font-medium transition-colors';
                btnDraw.className = 'px-3 py-1.5 text-sm rounded-md bg-gray-100 text-gray-600 hover:bg-gray-200 transition-colors';
                
                document.getElementById('preview-signature').style.display = '';
                const img = document.getElementById('preview-signature-img');
                if (img) img.style.display = 'none';
                updatePreview();
            } else {
                textPanel.classList.add('hidden');
                drawPanel.classList.remove('hidden');
                btnDraw.className = 'px-3 py-1.5 text-sm rounded-md bg-blue-100 text-blue-700 font-medium transition-colors';
                btnText.className = 'px-3 py-1.5 text-sm rounded-md bg-gray-100 text-gray-600 hover:bg-gray-200 transition-colors';
            }
        }

        // ==================== 实时预览 ====================
        const fields = [
            { input: 'input-name', preview: 'preview-name' },
            { input: 'input-birth', preview: 'preview-birth' },
            { input: 'input-position', preview: 'preview-position' },
            { input: 'input-email', preview: 'preview-email' },
            { input: 'input-phone', preview: 'preview-phone' },
            { input: 'input-idcard', preview: 'preview-idcard' },
            { input: 'input-signature', preview: 'preview-signature' },
        ];

        function updatePreview() {
            if (signMode !== 'text') return;
            fields.forEach(field => {
                const input = document.getElementById(field.input);
                const preview = document.getElementById(field.preview);
                if (input && preview) preview.textContent = input.value || '';
            });

            const lastday = document.getElementById('input-lastday').value;
            if (lastday) {
                const d = new Date(lastday);
                document.getElementById('preview-lastday').textContent = `${d.getFullYear()}年${d.getMonth() + 1}月${d.getDate()}日`;
            } else {
                document.getElementById('preview-lastday').textContent = '';
            }

            const signDate = document.getElementById('input-sign-date').value;
            if (signDate) {
                const d = new Date(signDate);
                document.getElementById('preview-sign-date').textContent = `${d.getFullYear()}年${d.getMonth() + 1}月${d.getDate()}日`;
            } else {
                document.getElementById('preview-sign-date').textContent = '';
            }
        }

        document.querySelectorAll('input').forEach(input => {
            input.addEventListener('input', updatePreview);
            input.addEventListener('change', updatePreview);
        });

        // ==================== PDF 生成 ====================
        async function generatePDF() {
            const btn = document.querySelector('button[onclick="generatePDF()"]');
            const originalText = btn.innerHTML;
            btn.innerHTML = '⏳ 生成中...';
            btn.disabled = true;

            try {
                const previewSection = document.getElementById('preview-section');
                const wasHidden = previewSection.style.display === 'none';
                if (wasHidden) {
                    previewSection.style.display = 'block';
                    previewSection.style.position = 'fixed';
                    previewSection.style.left = '-9999px';
                }
                
                const { jsPDF } = window.jspdf;
                const element = document.getElementById('preview-area');
                
                const canvas = await html2canvas(element, {
                    scale: 2,
                    useCORS: true,
                    allowTaint: true,
                    backgroundColor: '#ffffff',
                    logging: false
                });

                if (wasHidden) {
                    previewSection.style.display = 'none';
                    previewSection.style.position = '';
                    previewSection.style.left = '';
                }

                const imgData = canvas.toDataURL('image/png');
                const pdf = new jsPDF('p', 'mm', 'a4');
                const pdfWidth = pdf.internal.pageSize.getWidth();
                const pdfHeight = pdf.internal.pageSize.getHeight();
                const imgWidth = canvas.width;
                const imgHeight = canvas.height;
                const ratio = Math.min(pdfWidth / imgWidth, pdfHeight / imgHeight);
                
                pdf.addImage(imgData, 'PNG', (pdfWidth - imgWidth * ratio) / 2, 0, imgWidth * ratio, imgHeight * ratio);
                
                const name = document.getElementById('input-name').value || '离职申请表';
                pdf.save(`${name}_离职申请表.pdf`);
                showToast('✅ PDF 已生成并下载');

            } catch (error) {
                console.error('PDF生成失败:', error);
                alert('PDF生成失败，请尝试使用浏览器打印功能（Ctrl+P / Cmd+P），选择"另存为PDF"');
            } finally {
                btn.innerHTML = originalText;
                btn.disabled = false;
            }
        }

        // ==================== 初始化 ====================
        initScanGuide();
        detectDevice();
        checkUrlHash();
        window.addEventListener('resize', detectDevice);
    </script>
</body>
</html>
