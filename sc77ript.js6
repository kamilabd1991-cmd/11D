document.addEventListener('contextmenu', event => event.preventDefault());

const firebaseConfig = {
    apiKey: "AIzaSyCZlIbRIbQmi6ktmDdlJdxZz4pu-L-Jj9s",
    authDomain: "d10x-37f8c.firebaseapp.com",
    projectId: "d10x-37f8c",
    storageBucket: "d10x-37f8c.appspot.com",
    messagingSenderId: "1037956330902",
    appId: "1:1037956330902:web:04f9ec2e311ca56f218360",
    measurementId: "G-WD2MTCN3L5"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

const collections = {
    materials: db.collection('materials'),
    customers: db.collection('customers')
};

let materials = [];
let customers = [];

async function loadLocalData() {
    materials = (await localforage.getItem('materials')) || [];
    customers = (await localforage.getItem('customers')) || [];
}

async function saveLocalData() {
    await localforage.setItem('materials', materials);
    await localforage.setItem('customers', customers);
}

async function syncData() {
    if (!navigator.onLine) return;

    let queue = (await localforage.getItem('syncQueue')) || [];
    for (let action of queue) {
        try {
            if (action.type === 'add' || action.type === 'update') {
                await collections[action.collection].doc(action.id.toString()).set(action.data);
            } else if (action.type === 'delete') {
                await collections[action.collection].doc(action.id.toString()).delete();
            }
        } catch (e) {}
    }
    await localforage.setItem('syncQueue', []);

    try {
        const materialsSnap = await collections.materials.get();
        const serverMaterials = materialsSnap.docs.map(doc => doc.data());

        const customersSnap = await collections.customers.get();
        const serverCustomers = customersSnap.docs.map(doc => doc.data());

        materials = mergeById(materials, serverMaterials);
        customers = mergeById(customers, serverCustomers);

        await saveLocalData();
        renderMaterials();
        renderCustomers();
    } catch (e) {}
}

function mergeById(localArr, serverArr) {
    const map = {};
    [...serverArr, ...localArr].forEach(item => {
        map[item.id] = item;
    });
    return Object.values(map);
}

async function enqueueSync(action) {
    let queue = (await localforage.getItem('syncQueue')) || [];
    queue.push(action);
    await localforage.setItem('syncQueue', queue);
}

window.addEventListener('online', syncData);

function openTab(tabName, buttonEl) {
    document.querySelectorAll('.tab-content').forEach(tab => tab.classList.remove('active'));
    document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
    document.getElementById(tabName).classList.add('active');
    buttonEl.classList.add('active');
}

function showPasswordModal() {
    document.getElementById('passwordModal').style.display = 'block';
    document.getElementById('appContent').style.display = 'none';
}
function hidePasswordModal() {
    document.getElementById('passwordModal').style.display = 'none';
    document.getElementById('appContent').style.display = 'block';
}

function renderMaterials() {
    let materialsList = document.getElementById('materialsList');
    materialsList.innerHTML = '';
    materials.forEach((material, index) => {
        let div = document.createElement('div');
        div.className = 'item-card';
        div.innerHTML = `
            <div class="card-header">
                <span>${material.name}</span>
                <div class="card-actions">
                    <button class="icon-btn" onclick="editMaterial(${index})">✏️</button>
                    <button class="icon-btn delete-btn" onclick="deleteMaterial(${index})">🗑️</button>
                </div>
            </div>
            <div class="card-content">
                <p><strong>سعر الشراء:</strong> ${material.buyPrice.toLocaleString()} دينار</p>
                <p><strong>سعر البيع نقدًا:</strong> ${material.cashPrice.toLocaleString()} دينار</p>
                <p><strong>قسط 10 أشهر:</strong> ${material.install10.toLocaleString()} دينار</p>
                <p><strong>قسط 12 شهر:</strong> ${material.install12.toLocaleString()} دينار</p>
            </div>
        `;
        materialsList.appendChild(div);
    });
}

function showModal(options) {
    return new Promise((resolve) => {
        const modalOverlay = document.getElementById('modalOverlay');
        const modalTitle = document.getElementById('modalTitle');
        const modalMessage = document.getElementById('modalMessage');
        const modalInput = document.getElementById('modalInput');
        const modalButtons = document.getElementById('modalButtons');

        modalTitle.innerText = options.title || '';
        modalMessage.innerText = options.message || '';
        modalInput.style.display = 'none';
        modalInput.type = 'text';
        modalInput.value = '';
        modalButtons.innerHTML = '';

        if (options.type === 'alert') {
            const btnOk = document.createElement('button');
            btnOk.className = 'btn-modal btn-primary';
            btnOk.innerText = 'حسناً';
            btnOk.onclick = () => {
                modalOverlay.style.display = 'none';
                resolve(true);
            };
            modalButtons.appendChild(btnOk);
        }
        else if (options.type === 'confirm') {
            const btnCancel = document.createElement('button');
            btnCancel.className = 'btn-modal btn-secondary';
            btnCancel.innerText = 'إلغاء';
            btnCancel.onclick = () => {
                modalOverlay.style.display = 'none';
                resolve(false);
            };

            const btnOk = document.createElement('button');
            btnOk.className = 'btn-modal btn-primary';
            btnOk.innerText = 'تأكيد';
            btnOk.onclick = () => {
                modalOverlay.style.display = 'none';
                resolve(true);
            };

            modalButtons.appendChild(btnCancel);
            modalButtons.appendChild(btnOk);
        }
        else if (options.type === 'prompt') {
            modalInput.style.display = 'block';
            modalInput.placeholder = options.placeholder || '';
            const btnCancel = document.createElement('button');
            btnCancel.className = 'btn-modal btn-secondary';
            btnCancel.innerText = 'إلغاء';
            btnCancel.onclick = () => {
                modalOverlay.style.display = 'none';
                resolve(null);
            };

            const btnOk = document.createElement('button');
            btnOk.className = 'btn-modal btn-primary';
            btnOk.innerText = 'موافق';
            btnOk.onclick = () => {
                const val = modalInput.value.trim();
                modalOverlay.style.display = 'none';
                resolve(val);
            };

            modalButtons.appendChild(btnCancel);
            modalButtons.appendChild(btnOk);
        }
        else if (options.type === 'password_prompt') {
            modalInput.style.display = 'block';
            modalInput.type = 'password';
            modalInput.placeholder = 'أدخل كلمة المرور';
            const btnOk = document.createElement('button');
            btnOk.className = 'btn-modal btn-primary';
            btnOk.innerText = 'دخول';
            btnOk.onclick = () => {
                const val = modalInput.value.trim();
                modalOverlay.style.display = 'none';
                resolve(val);
            };
            modalButtons.appendChild(btnOk);
        }
        else if (options.type === 'edit_customer') {
            modalInput.style.display = 'block';
            modalInput.placeholder = 'اسم الزبون الجديد';
            modalInput.value = options.defaultValue || '';

            const btnCancel = document.createElement('button');
            btnCancel.className = 'btn-modal btn-secondary';
            btnCancel.innerText = 'إلغاء';
            btnCancel.onclick = () => {
                modalOverlay.style.display = 'none';
                resolve(null);
            };

            const btnSave = document.createElement('button');
            btnSave.className = 'btn-modal btn-primary';
            btnSave.innerText = 'حفظ';
            btnSave.onclick = () => {
                const val = modalInput.value.trim();
                if (!val) return;
                modalOverlay.style.display = 'none';
                resolve(val);
            };

            modalButtons.appendChild(btnCancel);
            modalButtons.appendChild(btnSave);
        }
        else if (options.type === 'alert_with_print_receipt') {
            const btnClose = document.createElement('button');
            btnClose.className = 'btn-modal btn-secondary';
            btnClose.innerText = 'إغلاق';
            btnClose.onclick = () => {
                modalOverlay.style.display = 'none';
                resolve(true);
            };
            const btnPrint = document.createElement('button');
            btnPrint.className = 'btn-modal btn-primary';
            btnPrint.style.background = '#10b981';
            btnPrint.innerText = 'طباعة الوصل';
            btnPrint.onclick = () => {
                printReceipt(extraData.customerName, extraData.amount, extraData.date, extraData.notes, extraData.remainingDebt);
            };
            modalButtons.appendChild(btnClose);
            modalButtons.appendChild(btnPrint);
        }
        else if (type === 'alert_with_print_statement') {
            const btnClose = document.createElement('button');
            btnClose.className = 'btn-modal btn-secondary';
            btnClose.innerText = 'إغلاق';
            btnClose.onclick = () => {
                modalOverlay.style.display = 'none';
                resolve(true);
            };
            const btnPrint = document.createElement('button');
            btnPrint.className = 'btn-modal btn-primary';
            btnPrint.style.background = '#10b981';
            btnPrint.innerText = 'طباعة الكشف';
            btnPrint.onclick = () => {
                const c = customers.find(x => x.id === extraData.customerId);
                printStatement(c);
            };
            modalButtons.appendChild(btnClose);
            modalButtons.appendChild(btnPrint);
        }

        modalOverlay.style.display = 'flex';
        modalInput.focus();
    });
}

async function customAlert(message) {
    return await showModal({
        type: 'alert',
        title: 'تنبيه',
        message
    });
}
async function customConfirm(message) {
    return await showModal({
        type: 'confirm',
        title: 'تأكيد',
        message
    });
}
async function customPrompt(message, placeholder='') {
    return await showModal({
        type: 'prompt',
        title: 'إدخال',
        message,
        placeholder
    });
}

window.addMaterial = async function() {
    let name = document.getElementById('materialName').value.trim();
    let buyPrice = parseFloat(document.getElementById('buyPrice').value);
    let cashPrice = parseFloat(document.getElementById('cashPrice').value);
    let install10 = parseFloat(document.getElementById('install10').value);
    let install12 = parseFloat(document.getElementById('install12').value);

    if (!name || isNaN(buyPrice) || isNaN(cashPrice) || isNaN(install10) || isNaN(install12)) {
        await customAlert('يرجى ملء جميع الحقول الخاصة بالمواد');
        return;
    }

    let material = {
        id: Date.now(),
        name,
        buyPrice,
        cashPrice,
        install10,
        install12
    };
    materials.push(material);
    await saveLocalData();
    await enqueueSync({ type: 'add', collection: 'materials', id: material.id, data: material });
    renderMaterials();

    document.getElementById('materialName').value = '';
    document.getElementById('buyPrice').value = '';
    document.getElementById('cashPrice').value = '';
    document.getElementById('install10').value = '';
    document.getElementById('install12').value = '';

    await customAlert('تمت إضافة المادة بنجاح');
};

window.editMaterial = async function(index) {
    let material = materials[index];
    let name = await customPrompt('تعديل اسم المادة', material.name);
    if (!name) return;
    let buyPrice = await customPrompt('تعديل سعر الشراء', material.buyPrice);
    let cashPrice = await customPrompt('تعديل سعر البيع نقدًا', material.cashPrice);
    let install10 = await customPrompt('تعديل سعر 10 أشهر', material.install10);
    let install12 = await customPrompt('تعديل سعر 12 شهر', material.install12);

    material.name = name;
    material.buyPrice = parseFloat(buyPrice);
    material.cashPrice = parseFloat(cashPrice);
    material.install10 = parseFloat(install10);
    material.install12 = parseFloat(install12);

    await saveLocalData();
    await enqueueSync({ type: 'update', collection: 'materials', id: material.id, data: material });
    renderMaterials();
    await customAlert('تم تعديل المادة بنجاح');
};

window.deleteMaterial = async function(index) {
    if (await customConfirm('هل أنت متأكد من حذف هذه المادة؟')) {
        let deleted = materials.splice(index, 1)[0];
        await saveLocalData();
        await enqueueSync({ type: 'delete', collection: 'materials', id: deleted.id });
        renderMaterials();
        await customAlert('تم حذف المادة بنجاح');
    }
};

function renderCustomers() {
    let customersList = document.getElementById('customersList');
    customersList.innerHTML = '';
    customers.forEach((c, index) => {
        let latestPayment = c.transactions.filter(t => t.type === 'تسديد').slice(-1)[0];
        let totalPaid = c.transactions.filter(t => t.type === 'تسديد').reduce((sum, t) => sum + Number(t.amount), 0);
        let card = document.createElement('div');
        card.className = 'item-card';
        card.innerHTML = `
            <div class="card-header">
                <span>${c.name}</span>
                <div class="card-actions">
                    <button class="icon-btn" onclick="editCustomer(${index})">✏️</button>
                    <button class="icon-btn delete-btn" onclick="deleteCustomer(${index})">🗑️</button>
                </div>
            </div>
            <div class="card-content">
                <p><strong>رقم الهاتف:</strong> ${c.phone}</p>
                <p><strong>الرصيد الكلي:</strong> ${Number(c.totalDebt).toLocaleString()} دينار</p>
                <p><strong>المدفوع:</strong> ${totalPaid.toLocaleString()} دينار</p>
                <p><strong>آخر تسديد:</strong> ${latestPayment ? latestPayment.date : 'لا يوجد'}</p>
                <p><strong>ملاحظات:</strong> ${c.notes || '-'}</p>
                <div style="display:flex; gap:8px; flex-wrap:wrap; margin-top:10px;">
                    <button class="btn btn-primary" onclick="addPayment(${c.id})">تسديد</button>
                    <button class="btn btn-secondary" onclick="addDebt(${c.id})">دين جديد</button>
                    <button class="btn btn-primary" onclick="showStatement(${c.id})">كشف حساب</button>
                </div>
            </div>
        `;
        customersList.appendChild(card);
    });
}

window.addCustomer = async function() {
    let name = document.getElementById('customerName').value.trim();
    let totalDebt = parseFloat(document.getElementById('customerDebt').value);
    let notes = document.getElementById('customerNotes').value.trim();

    if (!name || isNaN(totalDebt)) {
        await customAlert('يرجى ملء اسم الزبون والمبلغ');
        return;
    }

    let customer = {
        id: Date.now(),
        name,
        phone: '-',
        totalDebt,
        notes,
        isLate: false,
        transactions: [
            {
                type: 'دين أولي',
                amount: totalDebt,
                date: new Date().toISOString().split('T')[0],
                notes: notes || ''
            }
        ]
    };

    customers.push(customer);
    await saveLocalData();
    await enqueueSync({ type: 'add', collection: 'customers', id: customer.id, data: customer });
    renderCustomers();

    document.getElementById('customerName').value = '';
    document.getElementById('customerDebt').value = '';
    document.getElementById('customerNotes').value = '';

    await customAlert('تمت إضافة الزبون بنجاح');
};

window.editCustomer = async function(index) {
    const customer = customers[index];
    const newName = await showModal({
        type: 'edit_customer',
        title: 'تعديل اسم الزبون',
        message: '',
        defaultValue: customer.name
    });

    if (newName && newName.trim() !== '') {
        customer.name = newName.trim();
        await saveLocalData();
        await enqueueSync({ type: 'update', collection: 'customers', id: customer.id, data: customer });
        renderCustomers();
        await customAlert("تم تعديل اسم الزبون بنجاح.");
    }
};

window.deleteCustomer = async function(index) {
    let pass = await showModal({
        type: 'password_prompt',
        title: 'حذف الزبون',
        message: ''
    });

    if (pass !== "1993") {
        await customAlert("كلمة المرور غير صحيحة، لا يمكن حذف الزبون.");
        return;
    }

    if (await customConfirm('هل أنت متأكد من حذف الزبون وجميع حركاته؟')) {
        let deleted = customers.splice(index, 1)[0];
        await saveLocalData();
        await enqueueSync({ type: 'delete', collection: 'customers', id: deleted.id });
        renderCustomers();
        await customAlert('تم حذف الزبون بنجاح');
    }
};

window.addPayment = async function(customerId) {
    let customer = customers.find(c => c.id === customerId);
    let amount = parseFloat(await customPrompt(`أدخل مبلغ التسديد للزبون: ${customer.name}`));
    if (isNaN(amount) || amount <= 0) return;

    let notes = await customPrompt('ملاحظات التسديد (اختياري)');
    let date = new Date().toISOString().split('T')[0];

    customer.totalDebt -= amount;
    if (customer.totalDebt < 0) customer.totalDebt = 0;
    customer.transactions.push({
        type: 'تسديد',
        amount,
        date,
        notes: notes || ''
    });

    await saveLocalData();
    await enqueueSync({ type: 'update', collection: 'customers', id: customer.id, data: customer });
    renderCustomers();

    await showModal({
        type: 'alert_with_print_receipt',
        title: 'تمت عملية التسديد بنجاح',
        message: `تم تسديد مبلغ ${amount.toLocaleString()} دينار للزبون ${customer.name}`,
        extraData: {
            customerName: customer.name,
            amount: amount,
            date: date,
            notes: notes || 'لا توجد',
            remainingDebt: customer.totalDebt
        }
    });
};

window.addDebt = async function(customerId) {
    let customer = customers.find(c => c.id === customerId);
    let amount = parseFloat(await customPrompt(`أدخل مبلغ الدين الجديد للزبون: ${customer.name}`));
    if (isNaN(amount) || amount <= 0) return;

    let notes = await customPrompt('ملاحظات الدين الجديد (اختياري)');
    let date = new Date().toISOString().split('T')[0];

    customer.totalDebt += amount;
    customer.transactions.push({
        type: 'دين جديد',
        amount,
        date,
        notes: notes || ''
    });

    await saveLocalData();
    await enqueueSync({ type: 'update', collection: 'customers', id: customer.id, data: customer });
    renderCustomers();
    await customAlert(`تمت إضافة دين جديد بقيمة ${amount.toLocaleString()} دينار للزبون ${customer.name}`);
};

window.showStatement = async function(customerId) {
    let customer = customers.find(c => c.id === customerId);
    let transText = customer.transactions.map((t, idx) => {
        return `${idx + 1}- [${t.date}] ${t.type}: ${Number(t.amount).toLocaleString()} دينار${t.notes ? ' - ملاحظات: ' + t.notes : ''}`;
    }).join('\n');

    let msg = `كشف حساب الزبون:\n\nالاسم: ${customer.name}\nرقم الهاتف: ${customer.phone}\nالرصيد الكلي المتبقي: ${Number(customer.totalDebt).toLocaleString()} دينار\n\nسجل العمليات:\n${transText || 'لا توجد عمليات مسجلة'}`;

    await showModal({
        type: 'alert_with_print_statement',
        title: 'كشف حساب',
        message: msg,
        extraData: {
            customerId: customer.id
        }
    });
};

window.reversePayment = async function(customerId, transactionIndex) {
    let customer = customers.find(c => c.id === customerId);
    let transaction = customer.transactions[transactionIndex];
    if (!transaction || transaction.type !== 'تسديد') {
        await customAlert("لا يمكن إلغاء هذه العملية.");
        return;
    }

    if (await customConfirm(`هل تريد إلغاء تسديد بقيمة ${Number(transaction.amount).toLocaleString()} دينار للزبون ${customer.name}؟`)) {
        customer.totalDebt += Number(transaction.amount);
        customer.transactions.splice(transactionIndex, 1);

        await saveLocalData();
        await enqueueSync({ type: 'update', collection: 'customers', id: customer.id, data: customer });
        renderCustomers();
        await customAlert("تم إلغاء التسديد بنجاح.");
    }
};

window.showLateCustomers = async function() {
    let lateCustomers = customers.filter(c => c.isLate);
    if (lateCustomers.length > 0) {
        let msg = lateCustomers.map(c =>
            `- ${c.name} | الرصيد: ${Number(c.totalDebt).toLocaleString()} دينار`
        ).join('\n');
        await customAlert(`الزبائن المتأخرون:\n\n${msg}`);
    } else {
        await customAlert("لا يوجد زبائن متأخرين حالياً.");
    }
};

window.printData = function(title, contentHTML, customStyles = '', bodyClass = '') {
    let printWindow = window.open('', '', 'height=600,width=800');
    printWindow.document.write('<html lang="ar" dir="rtl"><head><title>' + title + '</title>');
    printWindow.document.write('<link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;800&display=swap" rel="stylesheet">');
    printWindow.document.write('<style>');
    printWindow.document.write('body{font-family:"Tajawal",sans-serif; padding:20px; text-align:right; margin:0;}');
    printWindow.document.write('table{width:100%;border-collapse:collapse;margin-top:20px;}');
    printWindow.document.write('th,td{border:1px solid #ddd;padding:10px;text-align:right;}');
    printWindow.document.write('th{background-color:#f2f2f2;}');
    printWindow.document.write('.header{text-align:center;margin-bottom:20px;border-bottom:2px solid #000;padding-bottom:10px;}');
    printWindow.document.write('.footer{margin-top:40px; text-align:center; font-weight:bold;}');
    printWindow.document.write(customStyles || '');
    printWindow.document.write('</style>');
    printWindow.document.write('</head><body class="' + bodyClass + '">');
    printWindow.document.write(contentHTML);
    printWindow.document.write('</body></html>');
    printWindow.document.close();
    printWindow.focus();
    setTimeout(() => {
        printWindow.print();
        printWindow.close();
    }, 500);
};

window.printReceipt = function(customerName, amount, date, notes, remainingDebt) {
    const receiptNo = String(Date.now()).slice(-4);
    const now = new Date();
    const timeText = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: true });
    const content = `
        <div class="receipt-wrap">
            <div class="receipt-box">
                <div class="shop-box">
                    <div class="shop-name">مجمع كامل فون للتقسيط</div>
                    <div class="shop-desc">اجهزة كهربائية - اثاث منزلية - موبايلات</div>
                    <div class="shop-phones">
                        <span>0773 676 1213</span>
                        <span>0781 800 7750</span>
                    </div>
                </div>

                <div class="meta-row">
                    <div>تاريخ: <span>${date}</span></div>
                    <div>رقم الوصل <span>${receiptNo}</span></div>
                </div>

                <div class="customer-pill">اسم الزبون ${customerName}</div>

                <div class="notes-title">الملاحظات</div>

                <table class="receipt-table">
                    <thead>
                        <tr>
                            <th class="row-label"></th>
                            <th>دينار</th>
                            <th>دولار</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td class="row-label">الدين</td>
                            <td>${Number(Number(remainingDebt) + Number(amount)).toLocaleString()}</td>
                            <td></td>
                        </tr>
                        <tr>
                            <td class="row-label">الواصل</td>
                            <td>${Number(amount).toLocaleString()}</td>
                            <td></td>
                        </tr>
                        <tr>
                            <td class="row-label">المتبقي</td>
                            <td>${Number(remainingDebt).toLocaleString()}</td>
                            <td></td>
                        </tr>
                    </tbody>
                </table>

                <div class="receiver-line">المستلم: ................................</div>
                <div class="receipt-datetime">${date} ${timeText}</div>
            </div>
        </div>
    `;
    const styles = `
        @page { size: auto; margin: 8mm; }
        body.receipt-body{padding:0;background:#fff;text-align:center;}
        .receipt-wrap{display:flex;justify-content:center;align-items:flex-start;padding:0;margin:0;}
        .receipt-box{width:78mm;color:#111;padding:6mm 4mm 4mm 4mm;box-sizing:border-box;font-size:15px;line-height:1.35;}
        .shop-box{border:1px solid #777;padding:10px 8px 8px;margin-bottom:14px;}
        .shop-name{font-size:24px;font-weight:800;line-height:1.3;margin-bottom:4px;}
        .shop-desc{font-size:15px;font-weight:500;margin-bottom:10px;}
        .shop-phones{display:flex;justify-content:space-between;gap:8px;font-size:15px;direction:ltr;}
        .meta-row{display:flex;justify-content:space-between;align-items:center;font-size:15px;margin-bottom:12px;}
        .customer-pill{border:1px solid #777;border-radius:14px;padding:7px 10px;margin-bottom:12px;font-size:15px;font-weight:500;text-align:right;}
        .notes-title{font-size:16px;font-weight:700;text-align:right;margin-bottom:8px;}
        .receipt-table{width:100%;border-collapse:separate;border-spacing:0 8px;margin-top:0;direction:rtl;}
        .receipt-table th,.receipt-table td{border:1px solid #777;padding:9px 8px;text-align:center;font-size:15px;background:#fff;}
        .receipt-table thead th{background:#fff;border:none;padding:0 8px 6px;font-weight:700;}
        .receipt-table thead .row-label{border:none;}
        .receipt-table tbody td{border-radius:12px;}
        .receipt-table .row-label{width:24%;font-weight:700;}
        .receiver-line{margin-top:12px;text-align:right;font-size:15px;font-weight:700;}
        .receipt-datetime{margin-top:14px;text-align:center;font-size:14px;letter-spacing:1px;direction:ltr;}
    `;
    printData('وصل تسديد', content, styles, 'receipt-body');
};

window.printStatement = function(customer) {
    let transRows = customer.transactions.map(t => `<tr><td>${t.date}</td><td>${t.type}</td><td>${Number(t.amount).toLocaleString()}</td><td>${t.notes || '-'}</td></tr>`).join('');
    let content = `
        <p><strong>اسم الزبون:</strong> ${customer.name}</p>
        <p><strong>رقم الهاتف:</strong> ${customer.phone}</p>
        <p><strong>الرصيد الكلي المتبقي:</strong> ${Number(customer.totalDebt).toLocaleString()} دينار</p>
        <table>
            <thead>
                <tr>
                    <th>التاريخ</th>
                    <th>النوع</th>
                    <th>المبلغ (دينار)</th>
                    <th>ملاحظات</th>
                </tr>
            </thead>
            <tbody>
                ${transRows || '<tr><td colspan="4" style="text-align:center;">لا توجد عمليات مسجلة</td></tr>'}
            </tbody>
        </table>
    `;
    printData('كشف حساب / معاملات الزبون', content);
};

window.onload = async function() {
    document.querySelector('.sidebar').style.display = 'none';
    document.querySelector('.main-content').style.display = 'none';
    
    while(true) {
        let pass = await showModal({ type: 'password_prompt', message: '' });
        if (pass === "1991") {
            break;
        } else {
            await customAlert("كلمة المرور غير صحيحة");
        }
    }

    document.querySelector('.sidebar').style.display = 'flex';
    document.querySelector('.main-content').style.display = 'block';

    await loadLocalData();
    renderMaterials();
    renderCustomers();
    await syncData();

    if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('sw.js');
    }
};
