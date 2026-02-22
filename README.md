<!DOCTYPE html>
<html lang="en" class="scroll-smooth">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MediSupply - Hospital Equipment</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://js.stripe.com/v3/"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .glass-effect {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(10px);
        }
        .dark .glass-effect {
            background: rgba(17, 24, 39, 0.9);
        }
        .fade-in { animation: fadeIn 0.3s ease-in; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        
        /* Admin Sidebar */
        .admin-sidebar {
            transition: transform 0.3s ease;
        }
        
        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
        }
        ::-webkit-scrollbar-thumb {
            background: #888;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #555;
        }
    </style>
</head>
<body class="bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-gray-100 transition-colors duration-300">

    <!-- AUTHENTICATION MODAL -->
    <div id="authModal" class="fixed inset-0 z-50 hidden items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm">
        <div class="bg-white dark:bg-gray-800 rounded-2xl p-8 max-w-md w-full mx-4 shadow-2xl transform transition-all">
            <div class="text-center mb-6">
                <div class="w-16 h-16 bg-primary-600 rounded-full flex items-center justify-center text-white mx-auto mb-4">
                    <i class="fas fa-user text-2xl"></i>
                </div>
                <h2 class="text-2xl font-bold" id="authTitle">Sign In</h2>
                <p class="text-gray-600 dark:text-gray-400 text-sm mt-2">Access your account or register</p>
            </div>
            
            <form id="authForm" onsubmit="handleAuth(event)" class="space-y-4">
                <div id="nameField" class="hidden">
                    <label class="block text-sm font-medium mb-1">Full Name</label>
                    <input type="text" id="userName" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">Email</label>
                    <input type="email" id="userEmail" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">Password</label>
                    <input type="password" id="userPassword" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                </div>
                <div id="adminCodeField" class="hidden">
                    <label class="block text-sm font-medium mb-1">Admin Code (Optional)</label>
                    <input type="text" id="adminCode" placeholder="Leave blank for customer account" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                    <p class="text-xs text-gray-500 mt-1">Enter admin code to create management account</p>
                </div>
                
                <button type="submit" class="w-full py-3 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-semibold transition-colors">
                    <span id="authButtonText">Sign In</span>
                </button>
            </form>
            
            <div class="mt-4 text-center text-sm">
                <button onclick="toggleAuthMode()" class="text-primary-600 hover:underline" id="authToggle">
                    Don't have an account? Register
                </button>
            </div>
            <button onclick="closeAuth()" class="absolute top-4 right-4 text-gray-400 hover:text-gray-600">
                <i class="fas fa-times text-xl"></i>
            </button>
        </div>
    </div>

    <!-- ADMIN PANEL (Hidden by default) -->
    <div id="adminPanel" class="hidden min-h-screen bg-gray-100 dark:bg-gray-900">
        <!-- Admin Sidebar -->
        <aside class="fixed left-0 top-0 h-full w-64 bg-white dark:bg-gray-800 border-r border-gray-200 dark:border-gray-700 z-40 transform -translate-x-full md:translate-x-0 transition-transform" id="adminSidebar">
            <div class="p-6 border-b border-gray-200 dark:border-gray-700">
                <div class="flex items-center gap-3">
                    <div class="w-10 h-10 bg-primary-600 rounded-lg flex items-center justify-center text-white">
                        <i class="fas fa-cog"></i>
                    </div>
                    <div>
                        <h2 class="font-bold text-lg">Admin Panel</h2>
                        <p class="text-xs text-gray-500" id="adminUserName">Manager</p>
                    </div>
                </div>
            </div>
            
            <nav class="p-4 space-y-2">
                <button onclick="showAdminSection('dashboard')" class="admin-nav-btn w-full text-left px-4 py-3 rounded-lg bg-primary-50 dark:bg-primary-900/20 text-primary-600 dark:text-primary-400 font-medium flex items-center gap-3">
                    <i class="fas fa-chart-line w-5"></i> Dashboard
                </button>
                <button onclick="showAdminSection('products')" class="admin-nav-btn w-full text-left px-4 py-3 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700 flex items-center gap-3 text-gray-700 dark:text-gray-300">
                    <i class="fas fa-box w-5"></i> Products
                </button>
                <button onclick="showAdminSection('categories')" class="admin-nav-btn w-full text-left px-4 py-3 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700 flex items-center gap-3 text-gray-700 dark:text-gray-300">
                    <i class="fas fa-tags w-5"></i> Categories
                </button>
                <button onclick="showAdminSection('orders')" class="admin-nav-btn w-full text-left px-4 py-3 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700 flex items-center gap-3 text-gray-700 dark:text-gray-300">
                    <i class="fas fa-shopping-bag w-5"></i> Orders
                </button>
                <button onclick="showAdminSection('chat')" class="admin-nav-btn w-full text-left px-4 py-3 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700 flex items-center gap-3 text-gray-700 dark:text-gray-300">
                    <i class="fas fa-comments w-5"></i> Live Chat
                    <span id="unreadBadge" class="ml-auto bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center hidden">0</span>
                </button>
                <button onclick="showAdminSection('settings')" class="admin-nav-btn w-full text-left px-4 py-3 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700 flex items-center gap-3 text-gray-700 dark:text-gray-300">
                    <i class="fas fa-cog w-5"></i> Settings
                </button>
                <hr class="my-4 border-gray-200 dark:border-gray-700">
                <button onclick="logout()" class="w-full text-left px-4 py-3 rounded-lg hover:bg-red-50 dark:hover:bg-red-900/20 text-red-600 flex items-center gap-3">
                    <i class="fas fa-sign-out-alt w-5"></i> Logout
                </button>
            </nav>
        </aside>

        <!-- Admin Content -->
        <main class="md:ml-64 min-h-screen p-8">
            <!-- Dashboard Section -->
            <div id="adminDashboard" class="admin-section fade-in">
                <h1 class="text-3xl font-bold mb-8">Dashboard Overview</h1>
                
                <div class="grid grid-cols-1 md:grid-cols-4 gap-6 mb-8">
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <div class="flex items-center justify-between mb-4">
                            <div class="w-12 h-12 bg-blue-100 dark:bg-blue-900/30 rounded-xl flex items-center justify-center text-blue-600">
                                <i class="fas fa-box text-xl"></i>
                            </div>
                            <span class="text-sm text-gray-500">Total Products</span>
                        </div>
                        <p class="text-3xl font-bold" id="totalProducts">0</p>
                    </div>
                    
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <div class="flex items-center justify-between mb-4">
                            <div class="w-12 h-12 bg-green-100 dark:bg-green-900/30 rounded-xl flex items-center justify-center text-green-600">
                                <i class="fas fa-dollar-sign text-xl"></i>
                            </div>
                            <span class="text-sm text-gray-500">Today's Sales</span>
                        </div>
                        <p class="text-3xl font-bold" id="todaySales">$0.00</p>
                    </div>
                    
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <div class="flex items-center justify-between mb-4">
                            <div class="w-12 h-12 bg-purple-100 dark:bg-purple-900/30 rounded-xl flex items-center justify-center text-purple-600">
                                <i class="fas fa-shopping-cart text-xl"></i>
                            </div>
                            <span class="text-sm text-gray-500">Pending Orders</span>
                        </div>
                        <p class="text-3xl font-bold" id="pendingOrders">0</p>
                    </div>
                    
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <div class="flex items-center justify-between mb-4">
                            <div class="w-12 h-12 bg-orange-100 dark:bg-orange-900/30 rounded-xl flex items-center justify-center text-orange-600">
                                <i class="fas fa-comments text-xl"></i>
                            </div>
                            <span class="text-sm text-gray-500">Active Chats</span>
                        </div>
                        <p class="text-3xl font-bold" id="activeChats">0</p>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <h3 class="font-bold text-lg mb-4">Recent Orders</h3>
                        <div id="recentOrdersList" class="space-y-3">
                            <p class="text-gray-500 text-center py-8">No recent orders</p>
                        </div>
                    </div>
                    
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <h3 class="font-bold text-lg mb-4">Low Stock Alerts</h3>
                        <div id="lowStockList" class="space-y-3">
                            <p class="text-gray-500 text-center py-8">No low stock items</p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Products Section -->
            <div id="adminProducts" class="admin-section hidden fade-in">
                <div class="flex justify-between items-center mb-8">
                    <h1 class="text-3xl font-bold">Manage Products</h1>
                    <button onclick="openProductEditor()" class="px-6 py-3 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-semibold flex items-center gap-2">
                        <i class="fas fa-plus"></i> Add Product
                    </button>
                </div>

                <div class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700 overflow-hidden">
                    <div class="p-4 border-b border-gray-200 dark:border-gray-700 flex gap-4">
                        <input type="text" id="adminProductSearch" placeholder="Search products..." oninput="searchAdminProducts()" class="flex-1 px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-gray-50 dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                        <select id="adminCategoryFilter" onchange="filterAdminProducts()" class="px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-gray-50 dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                            <option value="all">All Categories</option>
                        </select>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full">
                            <thead class="bg-gray-50 dark:bg-gray-900/50">
                                <tr>
                                    <th class="px-6 py-4 text-left text-sm font-semibold">Product</th>
                                    <th class="px-6 py-4 text-left text-sm font-semibold">Category</th>
                                    <th class="px-6 py-4 text-left text-sm font-semibold">Price</th>
                                    <th class="px-6 py-4 text-left text-sm font-semibold">Stock</th>
                                    <th class="px-6 py-4 text-left text-sm font-semibold">Actions</th>
                                </tr>
                            </thead>
                            <tbody id="adminProductsTable" class="divide-y divide-gray-200 dark:divide-gray-700">
                                <!-- Products injected here -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- Product Editor Modal -->
            <div id="productEditorModal" class="fixed inset-0 z-50 hidden items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm p-4">
                <div class="bg-white dark:bg-gray-800 rounded-2xl max-w-2xl w-full max-h-[90vh] overflow-y-auto shadow-2xl">
                    <div class="p-6 border-b border-gray-200 dark:border-gray-700 flex justify-between items-center">
                        <h2 class="text-2xl font-bold" id="editorTitle">Add New Product</h2>
                        <button onclick="closeProductEditor()" class="text-gray-400 hover:text-gray-600">
                            <i class="fas fa-times text-xl"></i>
                        </button>
                    </div>
                    <form id="productForm" onsubmit="saveProduct(event)" class="p-6 space-y-6">
                        <input type="hidden" id="editingProductId">
                        
                        <div class="grid grid-cols-2 gap-6">
                            <div class="col-span-2">
                                <label class="block text-sm font-medium mb-2">Product Name *</label>
                                <input type="text" id="prodName" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium mb-2">Price *</label>
                                <input type="number" id="prodPrice" step="0.01" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium mb-2">Stock Quantity *</label>
                                <input type="number" id="prodStock" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium mb-2">Category *</label>
                                <select id="prodCategory" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                                    <!-- Categories injected here -->
                                </select>
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium mb-2">Medical Function/Use</label>
                                <input type="text" id="prodFunction" placeholder="e.g., Cardiology, Emergency" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                            </div>
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium mb-2">Description *</label>
                            <textarea id="prodDescription" rows="3" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none"></textarea>
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium mb-2">Specifications (comma separated)</label>
                            <input type="text" id="prodSpecs" placeholder="Spec 1, Spec 2, Spec 3" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium mb-2">Product Image</label>
                            <div class="flex items-center gap-4">
                                <input type="file" id="prodImage" accept="image/*" onchange="previewImage(event)" class="hidden">
                                <button type="button" onclick="document.getElementById('prodImage').click()" class="px-4 py-2 border-2 border-dashed border-gray-300 dark:border-gray-700 rounded-lg hover:border-primary-500 transition-colors">
                                    <i class="fas fa-upload mr-2"></i> Upload Image
                                </button>
                                <img id="imagePreview" class="h-20 w-20 object-cover rounded-lg hidden">
                            </div>
                            <p class="text-xs text-gray-500 mt-2">Or paste image URL:</p>
                            <input type="text" id="prodImageUrl" placeholder="https://..." class="mt-1 w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none text-sm">
                        </div>
                        
                        <div class="flex gap-4 pt-4">
                            <button type="button" onclick="closeProductEditor()" class="flex-1 py-3 border border-gray-300 dark:border-gray-700 rounded-lg font-semibold hover:bg-gray-50 dark:hover:bg-gray-800 transition-colors">Cancel</button>
                            <button type="submit" class="flex-1 py-3 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-semibold transition-colors">Save Product</button>
                        </div>
                    </form>
                </div>
            </div>

            <!-- Categories Section -->
            <div id="adminCategories" class="admin-section hidden fade-in">
                <div class="flex justify-between items-center mb-8">
                    <h1 class="text-3xl font-bold">Categories & Functions</h1>
                    <button onclick="addCategory()" class="px-6 py-3 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-semibold flex items-center gap-2">
                        <i class="fas fa-plus"></i> Add Category
                    </button>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6" id="categoriesGrid">
                    <!-- Categories injected here -->
                </div>
            </div>

            <!-- Orders Section -->
            <div id="adminOrders" class="admin-section hidden fade-in">
                <h1 class="text-3xl font-bold mb-8">Order Management</h1>
                <div class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700 overflow-hidden">
                    <div class="p-4 border-b border-gray-200 dark:border-gray-700">
                        <div class="flex gap-4">
                            <button onclick="filterOrders('all')" class="px-4 py-2 rounded-lg bg-primary-600 text-white text-sm font-medium">All</button>
                            <button onclick="filterOrders('pending')" class="px-4 py-2 rounded-lg bg-gray-200 dark:bg-gray-700 text-gray-700 dark:text-gray-300 text-sm font-medium">Pending</button>
                            <button onclick="filterOrders('completed')" class="px-4 py-2 rounded-lg bg-gray-200 dark:bg-gray-700 text-gray-700 dark:text-gray-300 text-sm font-medium">Completed</button>
                        </div>
                    </div>
                    <div id="ordersList" class="divide-y divide-gray-200 dark:divide-gray-700">
                        <!-- Orders injected here -->
                    </div>
                </div>
            </div>

            <!-- Chat Section -->
            <div id="adminChat" class="admin-section hidden fade-in h-[calc(100vh-8rem)]">
                <h1 class="text-3xl font-bold mb-4">Customer Support Chat</h1>
                <div class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700 h-full flex">
                    <div class="w-1/3 border-r border-gray-200 dark:border-gray-700 overflow-y-auto">
                        <div id="chatSessionsList" class="divide-y divide-gray-200 dark:divide-gray-700">
                            <!-- Chat sessions -->
                        </div>
                    </div>
                    <div class="flex-1 flex flex-col">
                        <div id="adminChatMessages" class="flex-1 overflow-y-auto p-4 space-y-4">
                            <div class="text-center text-gray-500 mt-20">Select a conversation</div>
                        </div>
                        <div id="adminChatInput" class="p-4 border-t border-gray-200 dark:border-gray-700 hidden">
                            <form onsubmit="sendAdminMessage(event)" class="flex gap-2">
                                <input type="text" id="adminMessageInput" placeholder="Type your response..." class="flex-1 px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-gray-50 dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                                <button type="submit" class="px-6 py-2 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-medium">
                                    <i class="fas fa-paper-plane"></i>
                                </button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Settings Section -->
            <div id="adminSettings" class="admin-section hidden fade-in">
                <h1 class="text-3xl font-bold mb-8">Website Settings</h1>
                
                <div class="max-w-2xl space-y-6">
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <h3 class="font-bold text-lg mb-4">General Information</h3>
                        <div class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium mb-2">Website Name</label>
                                <input type="text" id="siteName" value="MediSupply" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-2">Support Email</label>
                                <input type="email" id="supportEmail" placeholder="support@yourdomain.com" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                                <p class="text-xs text-gray-500 mt-1">Customer inquiries will be sent here</p>
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-2">Contact Phone</label>
                                <input type="text" id="contactPhone" placeholder="+1 (555) 123-4567" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                            </div>
                        </div>
                    </div>

                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <h3 class="font-bold text-lg mb-4">Payment Methods</h3>
                        <div class="space-y-3">
                            <label class="flex items-center gap-3 p-3 border border-gray-200 dark:border-gray-700 rounded-lg cursor-pointer hover:bg-gray-50 dark:hover:bg-gray-900/50">
                                <input type="checkbox" id="enableStripe" class="w-5 h-5 text-primary-600 rounded focus:ring-primary-500">
                                <div class="flex-1">
                                    <div class="font-medium">Credit Cards (Stripe)</div>
                                    <div class="text-sm text-gray-500">Accept Visa, Mastercard, Amex</div>
                                </div>
                                <i class="fab fa-cc-visa text-2xl text-gray-400"></i>
                                <i class="fab fa-cc-mastercard text-2xl text-gray-400"></i>
                            </label>
                            
                            <label class="flex items-center gap-3 p-3 border border-gray-200 dark:border-gray-700 rounded-lg cursor-pointer hover:bg-gray-50 dark:hover:bg-gray-900/50">
                                <input type="checkbox" id="enablePayPal" class="w-5 h-5 text-primary-600 rounded focus:ring-primary-500">
                                <div class="flex-1">
                                    <div class="font-medium">PayPal</div>
                                    <div class="text-sm text-gray-500">PayPal checkout integration</div>
                                </div>
                                <i class="fab fa-paypal text-2xl text-blue-600"></i>
                            </label>
                            
                            <label class="flex items-center gap-3 p-3 border border-gray-200 dark:border-gray-700 rounded-lg cursor-pointer hover:bg-gray-50 dark:hover:bg-gray-900/50">
                                <input type="checkbox" id="enableBankTransfer" checked class="w-5 h-5 text-primary-600 rounded focus:ring-primary-500">
                                <div class="flex-1">
                                    <div class="font-medium">Bank Transfer</div>
                                    <div class="text-sm text-gray-500">Manual verification required</div>
                                </div>
                                <i class="fas fa-university text-2xl text-gray-400"></i>
                            </label>
                            
                            <label class="flex items-center gap-3 p-3 border border-gray-200 dark:border-gray-700 rounded-lg cursor-pointer hover:bg-gray-50 dark:hover:bg-gray-900/50">
                                <input type="checkbox" id="enableCOD" class="w-5 h-5 text-primary-600 rounded focus:ring-primary-500">
                                <div class="flex-1">
                                    <div class="font-medium">Cash on Delivery</div>
                                    <div class="text-sm text-gray-500">Pay when you receive</div>
                                </div>
                                <i class="fas fa-money-bill-wave text-2xl text-green-600"></i>
                            </label>
                        </div>
                        
                        <div class="mt-4 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg">
                            <p class="text-sm text-blue-800 dark:text-blue-200">
                                <i class="fas fa-info-circle mr-2"></i>
                                To enable live payments, add your Stripe/PayPal API keys in the backend configuration.
                            </p>
                        </div>
                    </div>

                    <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                        <h3 class="font-bold text-lg mb-4">Admin Access</h3>
                        <div class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium mb-2">Admin Registration Code</label>
                                <input type="text" id="adminRegCode" value="ADMIN2024" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none font-mono">
                                <p class="text-xs text-gray-500 mt-1">Share this code with managers to create admin accounts</p>
                            </div>
                        </div>
                    </div>

                    <button onclick="saveSettings()" class="w-full py-3 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-semibold transition-colors">
                        Save All Settings
                    </button>
                </div>
            </div>
        </main>
    </div>

    <!-- MAIN STORE (Visible to customers) -->
    <div id="storeFront">
        <!-- Navigation -->
        <nav class="fixed w-full z-30 glass-effect border-b border-gray-200 dark:border-gray-800 transition-colors duration-300">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="flex justify-between h-16 items-center">
                    <div class="flex items-center cursor-pointer" onclick="window.scrollTo(0,0)">
                        <div class="w-10 h-10 bg-primary-600 rounded-lg flex items-center justify-center text-white mr-3">
                            <i class="fas fa-heartbeat text-xl"></i>
                        </div>
                        <span class="text-xl font-bold tracking-tight" id="storeName">MediSupply</span>
                    </div>

                    <div class="hidden md:flex flex-1 max-w-lg mx-8">
                        <div class="relative w-full">
                            <input type="text" id="searchInput" placeholder="Search equipment..." 
                                class="w-full pl-10 pr-4 py-2 rounded-full border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-primary-500 transition-all">
                            <i class="fas fa-search absolute left-3 top-3 text-gray-400"></i>
                        </div>
                    </div>

                    <div class="flex items-center space-x-4">
                        <button onclick="toggleTheme()" class="p-2 rounded-lg hover:bg-gray-200 dark:hover:bg-gray-800 transition-colors">
                            <i class="fas fa-sun hidden dark:block text-yellow-400"></i>
                            <i class="fas fa-moon block dark:hidden text-gray-600"></i>
                        </button>

                        <button onclick="toggleCart()" class="relative p-2 rounded-lg hover:bg-gray-200 dark:hover:bg-gray-800 transition-colors">
                            <i class="fas fa-shopping-cart text-lg"></i>
                            <span id="cartBadge" class="absolute -top-1 -right-1 bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center hidden">0</span>
                        </button>

                        <button onclick="openAuth()" class="p-2 rounded-lg hover:bg-gray-200 dark:hover:bg-gray-800 transition-colors" id="authButton">
                            <i class="fas fa-user text-lg"></i>
                        </button>

                        <div class="hidden md:flex items-center gap-2" id="userDisplay">
                            <!-- User info injected here -->
                        </div>

                        <button onclick="toggleMobileMenu()" class="md:hidden p-2 rounded-lg hover:bg-gray-200 dark:hover:bg-gray-800">
                            <i class="fas fa-bars text-lg"></i>
                        </button>
                    </div>
                </div>
            </div>
        </nav>

        <!-- Hero Section -->
        <section class="pt-24 pb-12 px-4 sm:px-6 lg:px-8 max-w-7xl mx-auto">
            <div class="text-center animate-fade-in">
                <h1 class="text-4xl md:text-6xl font-bold mb-4 bg-gradient-to-r from-primary-600 to-cyan-500 bg-clip-text text-transparent">
                    Premium Hospital Equipment
                </h1>
                <p class="text-lg text-gray-600 dark:text-gray-400 mb-8 max-w-2xl mx-auto">
                    Reliable medical supplies and equipment for healthcare professionals. Quality assured, delivered to your facility.
                </p>
                
                <!-- Categories -->
                <div class="flex flex-wrap justify-center gap-3 mb-8" id="categoryFilters">
                    <button onclick="filterCategory('all')" class="category-btn px-6 py-2 rounded-full bg-primary-600 text-white font-medium transition-all hover:scale-105">All</button>
                </div>

                <!-- Function Filters -->
                <div class="flex flex-wrap justify-center gap-2 mb-8" id="functionFilters">
                    <!-- Function tags injected here -->
                </div>
            </div>
        </section>

        <!-- Products Grid -->
        <section class="px-4 sm:px-6 lg:px-8 max-w-7xl mx-auto pb-20">
            <div id="productsGrid" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
                <!-- Products injected here -->
            </div>
            <div id="noResults" class="hidden text-center py-20">
                <i class="fas fa-search text-6xl text-gray-300 dark:text-gray-700 mb-4"></i>
                <p class="text-xl text-gray-500 dark:text-gray-400">No equipment found matching your search.</p>
            </div>
        </section>

        <!-- Cart Sidebar -->
        <div id="cartSidebar" class="fixed inset-0 z-40 hidden">
            <div class="absolute inset-0 bg-black bg-opacity-50 backdrop-blur-sm" onclick="toggleCart()"></div>
            <div class="absolute right-0 top-0 h-full w-full max-w-md bg-white dark:bg-gray-900 shadow-2xl transform transition-transform duration-300 translate-x-full" id="cartContent">
                <div class="flex flex-col h-full">
                    <div class="flex items-center justify-between p-6 border-b border-gray-200 dark:border-gray-800">
                        <h2 class="text-2xl font-bold">Shopping Cart</h2>
                        <button onclick="toggleCart()" class="p-2 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg">
                            <i class="fas fa-times text-xl"></i>
                        </button>
                    </div>
                    
                    <div id="cartItems" class="flex-1 overflow-y-auto p-6 space-y-4">
                        <div class="text-center text-gray-500 dark:text-gray-400 mt-10">
                            <i class="fas fa-shopping-basket text-6xl mb-4 opacity-30"></i>
                            <p>Your cart is empty</p>
                        </div>
                    </div>

                    <div class="border-t border-gray-200 dark:border-gray-800 p-6 space-y-4">
                        <div class="flex justify-between text-lg font-semibold">
                            <span>Total</span>
                            <span id="cartTotal">$0.00</span>
                        </div>
                        
                        <!-- Payment Method Selection -->
                        <div class="space-y-2">
                            <label class="text-sm font-medium">Payment Method</label>
                            <select id="paymentMethod" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 focus:ring-2 focus:ring-primary-500 outline-none">
                                <option value="card">Credit Card (Stripe)</option>
                                <option value="paypal">PayPal</option>
                                <option value="bank">Bank Transfer</option>
                                <option value="cod">Cash on Delivery</option>
                            </select>
                        </div>
                        
                        <button onclick="checkout()" class="w-full py-3 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-semibold transition-colors flex items-center justify-center gap-2">
                            <span>Proceed to Checkout</span>
                            <i class="fas fa-arrow-right"></i>
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Product Modal -->
        <div id="productModal" class="fixed inset-0 z-40 hidden flex items-center justify-center p-4">
            <div class="absolute inset-0 bg-black bg-opacity-50 backdrop-blur-sm" onclick="closeProductModal()"></div>
            <div class="relative bg-white dark:bg-gray-900 rounded-2xl max-w-2xl w-full max-h-[90vh] overflow-y-auto animate-slide-up">
                <button onclick="closeProductModal()" class="absolute top-4 right-4 p-2 bg-gray-100 dark:bg-gray-800 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700 transition-colors z-10">
                    <i class="fas fa-times"></i>
                </button>
                <div id="modalContent"></div>
            </div>
        </div>

        <!-- Checkout Modal -->
        <div id="checkoutModal" class="fixed inset-0 z-50 hidden items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm p-4">
            <div class="bg-white dark:bg-gray-800 rounded-2xl max-w-lg w-full max-h-[90vh] overflow-y-auto shadow-2xl">
                <div class="p-6 border-b border-gray-200 dark:border-gray-700 flex justify-between items-center">
                    <h2 class="text-2xl font-bold">Complete Order</h2>
                    <button onclick="closeCheckout()" class="text-gray-400 hover:text-gray-600">
                        <i class="fas fa-times text-xl"></i>
                    </button>
                </div>
                <form id="checkoutForm" onsubmit="processOrder(event)" class="p-6 space-y-4">
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="block text-sm font-medium mb-1">First Name *</label>
                            <input type="text" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                        </div>
                        <div>
                            <label class="block text-sm font-medium mb-1">Last Name *</label>
                            <input type="text" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                        </div>
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Email *</label>
                        <input type="email" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Phone *</label>
                        <input type="tel" required class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Hospital/Facility Name</label>
                        <input type="text" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Shipping Address *</label>
                        <textarea required rows="2" class="w-full px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-900 focus:ring-2 focus:ring-primary-500 outline-none"></textarea>
                    </div>
                    
                    <div id="cardElement" class="hidden p-4 border-2 border-dashed border-gray-300 dark:border-gray-700 rounded-lg">
                        <p class="text-sm text-gray-500 text-center">Stripe Card Element would appear here</p>
                    </div>
                    
                    <div class="border-t border-gray-200 dark:border-gray-700 pt-4 mt-4">
                        <div class="flex justify-between text-lg font-bold mb-4">
                            <span>Total Amount:</span>
                            <span id="checkoutTotal">$0.00</span>
                        </div>
                        <button type="submit" class="w-full py-3 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-semibold transition-colors">
                            Place Order
                        </button>
                    </div>
                </form>
            </div>
        </div>

        <!-- Chat Widget -->
        <div class="fixed bottom-6 right-6 z-30">
            <button onclick="toggleChat()" class="w-14 h-14 bg-primary-600 hover:bg-primary-700 text-white rounded-full shadow-lg flex items-center justify-center transition-transform hover:scale-110 animate-bounce-slow">
                <i class="fas fa-comments text-2xl"></i>
            </button>
            <div id="chatWindow" class="absolute bottom-16 right-0 w-96 bg-white dark:bg-gray-900 rounded-2xl shadow-2xl border border-gray-200 dark:border-gray-800 hidden transform transition-all duration-300 origin-bottom-right">
                <div class="bg-primary-600 text-white p-4 rounded-t-2xl flex items-center justify-between">
                    <div class="flex items-center gap-3">
                        <div class="w-10 h-10 bg-white bg-opacity-20 rounded-full flex items-center justify-center">
                            <i class="fas fa-headset"></i>
                        </div>
                        <div>
                            <h3 class="font-semibold">Customer Support</h3>
                            <p class="text-xs text-primary-100 flex items-center gap-1">
                                <span class="w-2 h-2 bg-green-400 rounded-full animate-pulse"></span>
                                Online
                            </p>
                        </div>
                    </div>
                    <button onclick="toggleChat()" class="text-white hover:text-gray-200">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <div id="chatMessages" class="h-96 overflow-y-auto p-4 space-y-4 bg-gray-50 dark:bg-gray-950">
                    <div class="flex gap-3">
                        <div class="w-8 h-8 bg-primary-600 rounded-full flex items-center justify-center text-white text-sm flex-shrink-0">
                            <i class="fas fa-robot"></i>
                        </div>
                        <div class="bg-white dark:bg-gray-800 p-3 rounded-lg rounded-tl-none shadow-sm max-w-[80%]">
                            <p class="text-sm">Hello! Welcome to <span class="site-name">MediSupply</span>. How can I help you today?</p>
                            <span class="text-xs text-gray-400 mt-1 block">Just now</span>
                        </div>
                    </div>
                </div>
                <div class="p-4 border-t border-gray-200 dark:border-gray-800 bg-white dark:bg-gray-900 rounded-b-2xl">
                    <form onsubmit="sendMessage(event)" class="flex gap-2">
                        <input type="text" id="chatInput" placeholder="Type your message..." class="flex-1 px-4 py-2 rounded-full border border-gray-300 dark:border-gray-700 bg-gray-50 dark:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-primary-500 text-sm">
                        <button type="submit" class="w-10 h-10 bg-primary-600 hover:bg-primary-700 text-white rounded-full flex items-center justify-center transition-colors">
                            <i class="fas fa-paper-plane"></i>
                        </button>
                    </form>
                </div>
            </div>
        </div>

        <!-- Toast -->
        <div id="toast" class="fixed bottom-6 left-1/2 transform -translate-x-1/2 bg-gray-900 dark:bg-white text-white dark:text-gray-900 px-6 py-3 rounded-full shadow-lg transform translate-y-20 opacity-0 transition-all duration-300 z-50 flex items-center gap-3">
            <i class="fas fa-check-circle text-green-400 dark:text-green-600"></i>
            <span id="toastMessage">Added to cart</span>
        </div>

        <!-- Footer -->
        <footer class="bg-white dark:bg-gray-900 border-t border-gray-200 dark:border-gray-800 py-12">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="grid grid-cols-1 md:grid-cols-4 gap-8">
                    <div>
                        <div class="flex items-center mb-4">
                            <div class="w-8 h-8 bg-primary-600 rounded-lg flex items-center justify-center text-white mr-2">
                                <i class="fas fa-heartbeat"></i>
                            </div>
                            <span class="text-lg font-bold site-name">MediSupply</span>
                        </div>
                        <p class="text-gray-600 dark:text-gray-400 text-sm">Quality medical equipment for healthcare professionals worldwide.</p>
                    </div>
                    <div>
                        <h4 class="font-semibold mb-4">Categories</h4>
                        <ul class="space-y-2 text-sm text-gray-600 dark:text-gray-400" id="footerCategories">
                            <!-- Injected dynamically -->
                        </ul>
                    </div>
                    <div>
                        <h4 class="font-semibold mb-4">Support</h4>
                        <ul class="space-y-2 text-sm text-gray-600 dark:text-gray-400">
                            <li><a href="#" class="hover:text-primary-600">Contact Us</a></li>
                            <li><a href="#" class="hover:text-primary-600">Shipping Info</a></li>
                            <li><a href="#" class="hover:text-primary-600">Returns</a></li>
                            <li><a href="#" class="hover:text-primary-600">FAQ</a></li>
                        </ul>
                    </div>
                    <div>
                        <h4 class="font-semibold mb-4">Contact</h4>
                        <ul class="space-y-2 text-sm text-gray-600 dark:text-gray-400">
                            <li><i class="fas fa-envelope mr-2"></i> <span id="footerEmail">support@medisupply.com</span></li>
                            <li><i class="fas fa-phone mr-2"></i> <span id="footerPhone">+1 (555) 123-4567</span></li>
                            <li><i class="fas fa-map-marker-alt mr-2"></i> 123 Medical Ave, Health City</li>
                        </ul>
                    </div>
                </div>
                <div class="border-t border-gray-200 dark:border-gray-800 mt-8 pt-8 text-center text-sm text-gray-500">
                    © 2024 <span class="site-name">MediSupply</span>. All rights reserved.
                </div>
            </div>
        </footer>
    </div>

    <script>
        // DATA STORAGE
        let storeData = {
            products: [],
            categories: [
                { id: 'diagnostic', name: 'Diagnostic', icon: 'fa-stethoscope' },
                { id: 'surgical', name: 'Surgical', icon: 'fa-procedures' },
                { id: 'patient-care', name: 'Patient Care', icon: 'fa-bed' },
                { id: 'emergency', name: 'Emergency', icon: 'fa-ambulance' }
            ],
            functions: ['Cardiology', 'Orthopedics', 'General Surgery', 'Pediatrics', 'Emergency Care', 'Laboratory', 'Radiology'],
            orders: [],
            chats: {},
            settings: {
                siteName: 'MediSupply',
                supportEmail: 'support@medisupply.com',
                contactPhone: '+1 (555) 123-4567',
                adminCode: 'ADMIN2024',
                payments: {
                    stripe: false,
                    paypal: false,
                    bank: true,
                    cod: false
                }
            },
            users: [],
            currentUser: null
        };

        // Initialize default products if none exist
        const defaultProducts = [
            {
                id: 1,
                name: "Digital Blood Pressure Monitor",
                category: "diagnostic",
                function: "Cardiology",
                price: 89.99,
                image: "https://images.unsplash.com/photo-1631217868264-e5b90bb7e133?w=500&auto=format&fit=crop&q=60",
                description: "Professional-grade automatic blood pressure monitor with large LCD display.",
                specs: ["Automatic inflation", "Irregular heartbeat detection", "2x120 memory capacity"],
                stock: 15
            },
            {
                id: 2,
                name: "Surgical Instrument Set",
                category: "surgical",
                function: "General Surgery",
                price: 249.99,
                image: "https://images.unsplash.com/photo-1551076805-e1869033e561?w=500&auto=format&fit=crop&q=60",
                description: "Complete 20-piece surgical instrument set made from high-grade stainless steel.",
                specs: ["Stainless steel", "Autoclavable", "20 pieces", "Storage case included"],
                stock: 8
            },
            {
                id: 3,
                name: "Hospital Bed - Electric",
                category: "patient-care",
                function: "General Surgery",
                price: 1299.99,
                image: "https://images.unsplash.com/photo-1519494026892-80bbd2d6fd0d?w=500&auto=format&fit=crop&q=60",
                description: "Full electric hospital bed with remote control and side rails.",
                specs: ["Electric height adjustment", "Head and foot elevation", "Side rails"],
                stock: 5
            }
        ];

        // Load data from localStorage
        function loadData() {
            const saved = localStorage.getItem('medisupply_data');
            if (saved) {
                storeData = JSON.parse(saved);
            } else {
                storeData.products = [...defaultProducts];
                saveData();
            }
            updateUI();
        }

        function saveData() {
            localStorage.setItem('medisupply_data', JSON.stringify(storeData));
        }

        // STATE
        let cart = [];
        let currentCategory = 'all';
        let currentFunction = 'all';
        let chatOpen = false;
        let currentChatSession = null;
        let isAuthModeLogin = true;

        // INITIALIZATION
        document.addEventListener('DOMContentLoaded', () => {
            loadData();
            loadTheme();
            setupEventListeners();
            checkAuth();
        });

        function setupEventListeners() {
            document.getElementById('searchInput')?.addEventListener('input', handleSearch);
            document.getElementById('paymentMethod')?.addEventListener('change', handlePaymentChange);
        }

        // AUTHENTICATION
        function openAuth() {
            if (storeData.currentUser) {
                // Show user menu or logout
                if (confirm('Logout?')) logout();
                return;
            }
            document.getElementById('authModal').classList.remove('hidden');
            document.getElementById('authModal').classList.add('flex');
        }

        function closeAuth() {
            document.getElementById('authModal').classList.add('hidden');
            document.getElementById('authModal').classList.remove('flex');
        }

        function toggleAuthMode() {
            isAuthModeLogin = !isAuthModeLogin;
            document.getElementById('authTitle').textContent = isAuthModeLogin ? 'Sign In' : 'Create Account';
            document.getElementById('authButtonText').textContent = isAuthModeLogin ? 'Sign In' : 'Register';
            document.getElementById('authToggle').textContent = isAuthModeLogin ? "Don't have an account? Register" : "Already have an account? Sign In";
            document.getElementById('nameField').classList.toggle('hidden', isAuthModeLogin);
            document.getElementById('adminCodeField').classList.toggle('hidden', isAuthModeLogin);
        }

        function handleAuth(e) {
            e.preventDefault();
            const email = document.getElementById('userEmail').value;
            const password = document.getElementById('userPassword').value;
            
            if (isAuthModeLogin) {
                // Login
                const user = storeData.users.find(u => u.email === email && u.password === password);
                if (user) {
                    storeData.currentUser = user;
                    saveData();
                    closeAuth();
                    checkAuth();
                    showToast(`Welcome back, ${user.name}!`);
                } else {
                    alert('Invalid credentials');
                }
            } else {
                // Register
                const name = document.getElementById('userName').value;
                const adminCode = document.getElementById('adminCode').value;
                const isAdmin = adminCode === storeData.settings.adminCode;
                
                if (storeData.users.find(u => u.email === email)) {
                    alert('Email already registered');
                    return;
                }
                
                const newUser = {
                    id: Date.now(),
                    name,
                    email,
                    password,
                    isAdmin,
                    createdAt: new Date().toISOString()
                };
                
                storeData.users.push(newUser);
                storeData.currentUser = newUser;
                saveData();
                closeAuth();
                checkAuth();
                showToast(isAdmin ? 'Admin account created!' : 'Account created successfully!');
            }
        }

        function checkAuth() {
            const user = storeData.currentUser;
            const authButton = document.getElementById('authButton');
            const userDisplay = document.getElementById('userDisplay');
            
            if (user) {
                authButton.innerHTML = '<i class="fas fa-sign-out-alt text-lg"></i>';
                userDisplay.innerHTML = `
                    <span class="text-sm font-medium hidden lg:block">${user.name}</span>
                    ${user.isAdmin ? '<span class="text-xs bg-primary-600 text-white px-2 py-1 rounded">Admin</span>' : ''}
                `;
                userDisplay.classList.remove('hidden');
                
                if (user.isAdmin) {
                    // Show admin link
                    const nav = document.querySelector('nav .flex.items-center.space-x-4');
                    if (!document.getElementById('adminLink')) {
                        const adminLink = document.createElement('button');
                        adminLink.id = 'adminLink';
                        adminLink.className = 'hidden md:block px-3 py-1 bg-primary-600 text-white rounded-lg text-sm font-medium hover:bg-primary-700';
                        adminLink.textContent = 'Admin';
                        adminLink.onclick = openAdminPanel;
                        nav.insertBefore(adminLink, authButton);
                    }
                }
            } else {
                authButton.innerHTML = '<i class="fas fa-user text-lg"></i>';
                userDisplay.classList.add('hidden');
                const adminLink = document.getElementById('adminLink');
                if (adminLink) adminLink.remove();
            }
        }

        function logout() {
            storeData.currentUser = null;
            saveData();
            checkAuth();
            closeAdminPanel();
            showToast('Logged out successfully');
        }

        // ADMIN PANEL
        function openAdminPanel() {
            if (!storeData.currentUser?.isAdmin) {
                alert('Admin access required');
                return;
            }
            document.getElementById('storeFront').classList.add('hidden');
            document.getElementById('adminPanel').classList.remove('hidden');
            updateAdminDashboard();
            renderAdminProducts();
            renderCategories();
            renderOrders();
            updateChatSessions();
        }

        function closeAdminPanel() {
            document.getElementById('adminPanel').classList.add('hidden');
            document.getElementById('storeFront').classList.remove('hidden');
        }

        function showAdminSection(section) {
            document.querySelectorAll('.admin-section').forEach(s => s.classList.add('hidden'));
            document.getElementById('admin' + section.charAt(0).toUpperCase() + section.slice(1)).classList.remove('hidden');
            
            document.querySelectorAll('.admin-nav-btn').forEach(btn => {
                btn.classList.remove('bg-primary-50', 'dark:bg-primary-900/20', 'text-primary-600', 'dark:text-primary-400');
                btn.classList.add('text-gray-700', 'dark:text-gray-300');
            });
            event.target.closest('button').classList.add('bg-primary-50', 'dark:bg-primary-900/20', 'text-primary-600', 'dark:text-primary-400');
            event.target.closest('button').classList.remove('text-gray-700', 'dark:text-gray-300');
        }

        function updateAdminDashboard() {
            document.getElementById('totalProducts').textContent = storeData.products.length;
            document.getElementById('todaySales').textContent = '$' + calculateTodaySales().toFixed(2);
            document.getElementById('pendingOrders').textContent = storeData.orders.filter(o => o.status === 'pending').length;
            document.getElementById('activeChats').textContent = Object.keys(storeData.chats).length;
            
            // Recent orders
            const recentOrders = storeData.orders.slice(-5).reverse();
            const ordersHTML = recentOrders.length ? recentOrders.map(order => `
                <div class="flex items-center justify-between p-3 bg-gray-50 dark:bg-gray-900/50 rounded-lg">
                    <div>
                        <p class="font-medium text-sm">Order #${order.id}</p>
                        <p class="text-xs text-gray-500">${order.customer.name}</p>
                    </div>
                    <div class="text-right">
                        <p class="font-bold text-sm">$${order.total.toFixed(2)}</p>
                        <span class="text-xs px-2 py-1 rounded-full ${order.status === 'completed' ? 'bg-green-100 text-green-800' : 'bg-yellow-100 text-yellow-800'}">${order.status}</span>
                    </div>
                </div>
            `).join('') : '<p class="text-gray-500 text-center py-8">No recent orders</p>';
            document.getElementById('recentOrdersList').innerHTML = ordersHTML;
            
            // Low stock
            const lowStock = storeData.products.filter(p => p.stock < 10);
            const stockHTML = lowStock.length ? lowStock.map(p => `
                <div class="flex items-center justify-between p-3 bg-red-50 dark:bg-red-900/20 rounded-lg">
                    <div class="flex items-center gap-3">
                        <img src="${p.image}" class="w-10 h-10 object-cover rounded">
                        <div>
                            <p class="font-medium text-sm">${p.name}</p>
                            <p class="text-xs text-red-600">Only ${p.stock} left</p>
                        </div>
                    </div>
                    <button onclick="openProductEditor(${p.id})" class="text-sm text-primary-600 hover:underline">Restock</button>
                </div>
            `).join('') : '<p class="text-gray-500 text-center py-8">No low stock items</p>';
            document.getElementById('lowStockList').innerHTML = stockHTML;
        }

        function calculateTodaySales() {
            const today = new Date().toDateString();
            return storeData.orders
                .filter(o => new Date(o.date).toDateString() === today && o.status === 'completed')
                .reduce((sum, o) => sum + o.total, 0);
        }

        // PRODUCT MANAGEMENT
        function renderAdminProducts() {
            const tbody = document.getElementById('adminProductsTable');
            const filter = document.getElementById('adminCategoryFilter');
            
            // Update category filter
            filter.innerHTML = '<option value="all">All Categories</option>' + 
                storeData.categories.map(c => `<option value="${c.id}">${c.name}</option>`).join('');
            
            tbody.innerHTML = storeData.products.map(p => `
                <tr class="hover:bg-gray-50 dark:hover:bg-gray-900/50">
                    <td class="px-6 py-4">
                        <div class="flex items-center gap-3">
                            <img src="${p.image}" class="w-12 h-12 object-cover rounded-lg">
                            <div>
                                <p class="font-medium">${p.name}</p>
                                <p class="text-xs text-gray-500">${p.function || 'No function specified'}</p>
                            </div>
                        </div>
                    </td>
                    <td class="px-6 py-4 text-sm">${storeData.categories.find(c => c.id === p.category)?.name || p.category}</td>
                    <td class="px-6 py-4 font-medium">$${p.price.toFixed(2)}</td>
                    <td class="px-6 py-4">
                        <span class="${p.stock < 10 ? 'text-red-600 font-bold' : ''}">${p.stock}</span>
                    </td>
                    <td class="px-6 py-4">
                        <div class="flex gap-2">
                            <button onclick="openProductEditor(${p.id})" class="p-2 text-blue-600 hover:bg-blue-50 rounded-lg">
                                <i class="fas fa-edit"></i>
                            </button>
                            <button onclick="deleteProduct(${p.id})" class="p-2 text-red-600 hover:bg-red-50 rounded-lg">
                                <i class="fas fa-trash"></i>
                            </button>
                        </div>
                    </td>
                </tr>
            `).join('');
        }

        function openProductEditor(productId = null) {
            const modal = document.getElementById('productEditorModal');
            const form = document.getElementById('productForm');
            const categorySelect = document.getElementById('prodCategory');
            
            // Populate categories
            categorySelect.innerHTML = storeData.categories.map(c => 
                `<option value="${c.id}">${c.name}</option>`
            ).join('');
            
            if (productId) {
                const product = storeData.products.find(p => p.id === productId);
                document.getElementById('editorTitle').textContent = 'Edit Product';
                document.getElementById('editingProductId').value = product.id;
                document.getElementById('prodName').value = product.name;
                document.getElementById('prodPrice').value = product.price;
                document.getElementById('prodStock').value = product.stock;
                document.getElementById('prodCategory').value = product.category;
                document.getElementById('prodFunction').value = product.function || '';
                document.getElementById('prodDescription').value = product.description;
                document.getElementById('prodSpecs').value = product.specs.join(', ');
                document.getElementById('prodImageUrl').value = product.image;
                
                const preview = document.getElementById('imagePreview');
                preview.src = product.image;
                preview.classList.remove('hidden');
            } else {
                document.getElementById('editorTitle').textContent = 'Add New Product';
                form.reset();
                document.getElementById('editingProductId').value = '';
                document.getElementById('imagePreview').classList.add('hidden');
            }
            
            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function closeProductEditor() {
            const modal = document.getElementById('productEditorModal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
        }

        function previewImage(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const preview = document.getElementById('imagePreview');
                    preview.src = e.target.result;
                    preview.classList.remove('hidden');
                    document.getElementById('prodImageUrl').value = e.target.result;
                };
                reader.readAsDataURL(file);
            }
        }

        function saveProduct(e) {
            e.preventDefault();
            
            const productId = document.getElementById('editingProductId').value;
            const productData = {
                name: document.getElementById('prodName').value,
                price: parseFloat(document.getElementById('prodPrice').value),
                stock: parseInt(document.getElementById('prodStock').value),
                category: document.getElementById('prodCategory').value,
                function: document.getElementById('prodFunction').value,
                description: document.getElementById('prodDescription').value,
                specs: document.getElementById('prodSpecs').value.split(',').map(s => s.trim()).filter(s => s),
                image: document.getElementById('prodImageUrl').value || 'https://via.placeholder.com/500'
            };
            
            if (productId) {
                // Edit existing
                const index = storeData.products.findIndex(p => p.id == productId);
                storeData.products[index] = { ...storeData.products[index], ...productData };
                showToast('Product updated successfully');
            } else {
                // Add new
                const newProduct = {
                    id: Date.now(),
                    ...productData
                };
                storeData.products.push(newProduct);
                showToast('Product added successfully');
            }
            
            saveData();
            closeProductEditor();
            renderAdminProducts();
            updateUI();
        }

        function deleteProduct(productId) {
            if (confirm('Are you sure you want to delete this product?')) {
                storeData.products = storeData.products.filter(p => p.id !== productId);
                saveData();
                renderAdminProducts();
                updateUI();
                showToast('Product deleted');
            }
        }

        function searchAdminProducts() {
            const query = document.getElementById('adminProductSearch').value.toLowerCase();
            const filtered = storeData.products.filter(p => 
                p.name.toLowerCase().includes(query) || 
                p.description.toLowerCase().includes(query)
            );
            // Re-render table with filtered results
            const tbody = document.getElementById('adminProductsTable');
            tbody.innerHTML = filtered.map(p => `
                <tr class="hover:bg-gray-50 dark:hover:bg-gray-900/50">
                    <td class="px-6 py-4">
                        <div class="flex items-center gap-3">
                            <img src="${p.image}" class="w-12 h-12 object-cover rounded-lg">
                            <span class="font-medium">${p.name}</span>
                        </div>
                    </td>
                    <td class="px-6 py-4 text-sm">${storeData.categories.find(c => c.id === p.category)?.name}</td>
                    <td class="px-6 py-4 font-medium">$${p.price.toFixed(2)}</td>
                    <td class="px-6 py-4">${p.stock}</td>
                    <td class="px-6 py-4">
                        <button onclick="openProductEditor(${p.id})" class="p-2 text-blue-600 hover:bg-blue-50 rounded-lg mr-2">
                            <i class="fas fa-edit"></i>
                        </button>
                        <button onclick="deleteProduct(${p.id})" class="p-2 text-red-600 hover:bg-red-50 rounded-lg">
                            <i class="fas fa-trash"></i>
                        </button>
                    </td>
                </tr>
            `).join('');
        }

        function filterAdminProducts() {
            const category = document.getElementById('adminCategoryFilter').value;
            if (category === 'all') {
                renderAdminProducts();
            } else {
                const filtered = storeData.products.filter(p => p.category === category);
                // Render filtered...
            }
        }

        // CATEGORIES
        function renderCategories() {
            const grid = document.getElementById('categoriesGrid');
            grid.innerHTML = storeData.categories.map(c => `
                <div class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-gray-700">
                    <div class="flex items-center justify-between mb-4">
                        <div class="w-12 h-12 bg-primary-100 dark:bg-primary-900/30 rounded-xl flex items-center justify-center text-primary-600 text-xl">
                            <i class="fas ${c.icon}"></i>
                        </div>
                        <button onclick="deleteCategory('${c.id}')" class="text-red-500 hover:text-red-600">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                    <h3 class="font-bold text-lg">${c.name}</h3>
                    <p class="text-sm text-gray-500 mt-1">${storeData.products.filter(p => p.category === c.id).length} products</p>
                </div>
            `).join('') + `
                <button onclick="addCategory()" class="border-2 border-dashed border-gray-300 dark:border-gray-700 rounded-2xl p-6 flex flex-col items-center justify-center text-gray-500 hover:border-primary-500 hover:text-primary-600 transition-colors min-h-[160px]">
                    <i class="fas fa-plus text-3xl mb-2"></i>
                    <span class="font-medium">Add Category</span>
                </button>
            `;
        }

        function addCategory() {
            const name = prompt('Enter category name:');
            if (name) {
                const id = name.toLowerCase().replace(/\s+/g, '-');
                const icon = 'fa-tag'; // Default icon
                storeData.categories.push({ id, name, icon });
                saveData();
                renderCategories();
                updateUI();
            }
        }

        function deleteCategory(categoryId) {
            if (confirm('Delete this category? Products in this category will become uncategorized.')) {
                storeData.categories = storeData.categories.filter(c => c.id !== categoryId);
                saveData();
                renderCategories();
                updateUI();
            }
        }

        // ORDERS
        function renderOrders() {
            const container = document.getElementById('ordersList');
            if (!storeData.orders.length) {
                container.innerHTML = '<p class="p-8 text-center text-gray-500">No orders yet</p>';
                return;
            }
            
            container.innerHTML = storeData.orders.slice().reverse().map(order => `
                <div class="p-6 hover:bg-gray-50 dark:hover:bg-gray-900/50 transition-colors">
                    <div class="flex items-center justify-between mb-4">
                        <div>
                            <h4 class="font-bold text-lg">Order #${order.id}</h4>
                            <p class="text-sm text-gray-500">${new Date(order.date).toLocaleString()}</p>
                        </div>
                        <div class="flex items-center gap-4">
                            <span class="px-3 py-1 rounded-full text-sm font-medium ${
                                order.status === 'completed' ? 'bg-green-100 text-green-800' : 
                                order.status === 'pending' ? 'bg-yellow-100 text-yellow-800' : 
                                'bg-gray-100 text-gray-800'
                            }">${order.status}</span>
                            <select onchange="updateOrderStatus(${order.id}, this.value)" class="text-sm border border-gray-300 dark:border-gray-700 rounded-lg px-3 py-1 bg-white dark:bg-gray-900">
                                <option value="pending" ${order.status === 'pending' ? 'selected' : ''}>Pending</option>
                                <option value="processing" ${order.status === 'processing' ? 'selected' : ''}>Processing</option>
                                <option value="completed" ${order.status === 'completed' ? 'selected' : ''}>Completed</option>
                                <option value="cancelled" ${order.status === 'cancelled' ? 'selected' : ''}>Cancelled</option>
                            </select>
                        </div>
                    </div>
                    <div class="space-y-2 mb-4">
                        ${order.items.map(item => `
                            <div class="flex items-center gap-3 text-sm">
                                <img src="${item.image}" class="w-10 h-10 object-cover rounded">
                                <span class="flex-1">${item.name} x${item.quantity}</span>
                                <span class="font-medium">$${(item.price * item.quantity).toFixed(2)}</span>
                            </div>
                        `).join('')}
                    </div>
                    <div class="flex justify-between items-center pt-4 border-t border-gray-200 dark:border-gray-700">
                        <div class="text-sm">
                            <p><span class="font-medium">Customer:</span> ${order.customer.name}</p>
                            <p><span class="font-medium">Email:</span> ${order.customer.email}</p>
                            <p><span class="font-medium">Payment:</span> ${order.paymentMethod}</p>
                        </div>
                        <div class="text-right">
                            <p class="text-2xl font-bold">$${order.total.toFixed(2)}</p>
                        </div>
                    </div>
                </div>
            `).join('');
        }

        function updateOrderStatus(orderId, status) {
            const order = storeData.orders.find(o => o.id === orderId);
            if (order) {
                order.status = status;
                saveData();
                updateAdminDashboard();
            }
        }

        function filterOrders(status) {
            // Filter implementation
        }

        // CHAT SYSTEM
        function updateChatSessions() {
            const list = document.getElementById('chatSessionsList');
            const sessions = Object.entries(storeData.chats);
            
            if (!sessions.length) {
                list.innerHTML = '<p class="p-4 text-center text-gray-500">No active chats</p>';
                return;
            }
            
            list.innerHTML = sessions.map(([sessionId, chat]) => {
                const lastMessage = chat.messages[chat.messages.length - 1];
                const unread = chat.messages.filter(m => m.sender === 'user' && !m.read).length;
                
                return `
                    <div onclick="openChatSession('${sessionId}')" class="p-4 hover:bg-gray-50 dark:hover:bg-gray-900/50 cursor-pointer border-b border-gray-200 dark:border-gray-700 ${currentChatSession === sessionId ? 'bg-primary-50 dark:bg-primary-900/20' : ''}">
                        <div class="flex items-center justify-between mb-1">
                            <span class="font-medium">${chat.customerName || 'Guest'}</span>
                            ${unread ? `<span class="bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center">${unread}</span>` : ''}
                        </div>
                        <p class="text-sm text-gray-500 truncate">${lastMessage?.text || 'No messages'}</p>
                        <p class="text-xs text-gray-400 mt-1">${new Date(lastMessage?.time).toLocaleTimeString()}</p>
                    </div>
                `;
            }).join('');
            
            // Update badge
            const totalUnread = sessions.reduce((sum, [_, chat]) => 
                sum + chat.messages.filter(m => m.sender === 'user' && !m.read).length, 0
            );
            const badge = document.getElementById('unreadBadge');
            if (totalUnread > 0) {
                badge.textContent = totalUnread;
                badge.classList.remove('hidden');
            } else {
                badge.classList.add('hidden');
            }
        }

        function openChatSession(sessionId) {
            currentChatSession = sessionId;
            const chat = storeData.chats[sessionId];
            
            // Mark messages as read
            chat.messages.forEach(m => { if (m.sender === 'user') m.read = true; });
            saveData();
            updateChatSessions();
            
            // Render messages
            const container = document.getElementById('adminChatMessages');
            container.innerHTML = chat.messages.map(m => `
                <div class="flex gap-3 ${m.sender === 'admin' ? 'justify-end' : ''}">
                    ${m.sender === 'user' ? `
                        <div class="w-8 h-8 bg-gray-300 rounded-full flex items-center justify-center text-sm">
                            <i class="fas fa-user"></i>
                        </div>
                    ` : ''}
                    <div class="${m.sender === 'admin' ? 'bg-primary-600 text-white' : 'bg-white dark:bg-gray-800'} p-3 rounded-lg ${m.sender === 'admin' ? 'rounded-tr-none' : 'rounded-tl-none'} shadow-sm max-w-[70%]">
                        <p class="text-sm">${m.text}</p>
                        <span class="text-xs ${m.sender === 'admin' ? 'text-primary-100' : 'text-gray-400'} mt-1 block">${new Date(m.time).toLocaleTimeString()}</span>
                    </div>
                    ${m.sender === 'admin' ? `
                        <div class="w-8 h-8 bg-primary-600 rounded-full flex items-center justify-center text-white text-sm">
                            <i class="fas fa-headset"></i>
                        </div>
                    ` : ''}
                </div>
            `).join('');
            
            document.getElementById('adminChatInput').classList.remove('hidden');
            container.scrollTop = container.scrollHeight;
        }

        function sendAdminMessage(e) {
            e.preventDefault();
            if (!currentChatSession) return;
            
            const input = document.getElementById('adminMessageInput');
            const text = input.value.trim();
            if (!text) return;
            
            const chat = storeData.chats[currentChatSession];
            chat.messages.push({
                sender: 'admin',
                text,
                time: new Date().toISOString(),
                read: true
            });
            
            saveData();
            input.value = '';
            openChatSession(currentChatSession); // Refresh
        }

        // SETTINGS
        function saveSettings() {
            storeData.settings.siteName = document.getElementById('siteName').value;
            storeData.settings.supportEmail = document.getElementById('supportEmail').value;
            storeData.settings.contactPhone = document.getElementById('contactPhone').value;
            storeData.settings.adminCode = document.getElementById('adminRegCode').value;
            storeData.settings.payments = {
                stripe: document.getElementById('enableStripe').checked,
                paypal: document.getElementById('enablePayPal').checked,
                bank: document.getElementById('enableBankTransfer').checked,
                cod: document.getElementById('enableCOD').checked
            };
            
            saveData();
            updateUI();
            showToast('Settings saved successfully');
        }

        // STORE FRONT FUNCTIONS
        function updateUI() {
            // Update site name everywhere
            document.querySelectorAll('.site-name').forEach(el => el.textContent = storeData.settings.siteName);
            document.getElementById('storeName').textContent = storeData.settings.siteName;
            document.title = storeData.settings.siteName + ' - Hospital Equipment';
            
            // Update contact info
            document.getElementById('footerEmail').textContent = storeData.settings.supportEmail;
            document.getElementById('footerPhone').textContent = storeData.settings.contactPhone;
            
            // Render categories
            const catContainer = document.getElementById('categoryFilters');
            catContainer.innerHTML = `
                <button onclick="filterCategory('all')" class="category-btn px-6 py-2 rounded-full ${currentCategory === 'all' ? 'bg-primary-600 text-white' : 'bg-gray-200 dark:bg-gray-800 text-gray-700 dark:text-gray-300'} font-medium transition-all hover:scale-105">All</button>
            ` + storeData.categories.map(c => `
                <button onclick="filterCategory('${c.id}')" class="category-btn px-6 py-2 rounded-full ${currentCategory === c.id ? 'bg-primary-600 text-white' : 'bg-gray-200 dark:bg-gray-800 text-gray-700 dark:text-gray-300'} font-medium transition-all hover:scale-105">${c.name}</button>
            `).join('');
            
            // Render function filters
            const funcContainer = document.getElementById('functionFilters');
            funcContainer.innerHTML = storeData.functions.map(f => `
                <button onclick="filterFunction('${f}')" class="px-4 py-1 rounded-full text-sm ${currentFunction === f ? 'bg-cyan-600 text-white' : 'bg-gray-100 dark:bg-gray-800 text-gray-600 dark:text-gray-400'} transition-colors hover:bg-cyan-100 dark:hover:bg-cyan-900/30">${f}</button>
            `).join('');
            
            // Update footer categories
            document.getElementById('footerCategories').innerHTML = storeData.categories.map(c => 
                `<li><a href="#" onclick="filterCategory('${c.id}'); return false;" class="hover:text-primary-600">${c.name}</a></li>`
            ).join('');
            
            // Render products
            renderProducts();
            
            // Update payment options
            const paymentSelect = document.getElementById('paymentMethod');
            if (paymentSelect) {
                let options = [];
                if (storeData.settings.payments.stripe) options.push('<option value="card">Credit Card</option>');
                if (storeData.settings.payments.paypal) options.push('<option value="paypal">PayPal</option>');
                if (storeData.settings.payments.bank) options.push('<option value="bank">Bank Transfer</option>');
                if (storeData.settings.payments.cod) options.push('<option value="cod">Cash on Delivery</option>');
                paymentSelect.innerHTML = options.length ? options.join('') : '<option value="bank">Bank Transfer</option>';
            }
        }

        function renderProducts() {
            const grid = document.getElementById('productsGrid');
            const noResults = document.getElementById('noResults');
            
            let filtered = storeData.products;
            if (currentCategory !== 'all') {
                filtered = filtered.filter(p => p.category === currentCategory);
            }
            if (currentFunction !== 'all') {
                filtered = filtered.filter(p => p.function === currentFunction);
            }
            
            if (filtered.length === 0) {
                grid.innerHTML = '';
                noResults.classList.remove('hidden');
                return;
            }
            
            noResults.classList.add('hidden');
            grid.innerHTML = filtered.map(product => `
                <div class="group bg-white dark:bg-gray-800 rounded-2xl shadow-sm hover:shadow-xl transition-all duration-300 overflow-hidden border border-gray-100 dark:border-gray-700 flex flex-col">
                    <div class="relative overflow-hidden aspect-square bg-gray-100 dark:bg-gray-700">
                        <img src="${product.image}" alt="${product.name}" class="w-full h-full object-cover transform group-hover:scale-110 transition-transform duration-500">
                        ${product.stock < 10 ? `<span class="absolute top-3 left-3 bg-red-500 text-white text-xs font-bold px-3 py-1 rounded-full">Low Stock</span>` : ''}
                        ${product.function ? `<span class="absolute top-3 right-3 bg-cyan-600 text-white text-xs px-3 py-1 rounded-full">${product.function}</span>` : ''}
                        <button onclick="openProductModal(${product.id})" class="absolute bottom-3 right-3 w-10 h-10 bg-white dark:bg-gray-900 rounded-full shadow-lg flex items-center justify-center opacity-0 group-hover:opacity-100 transform translate-y-4 group-hover:translate-y-0 transition-all duration-300 hover:bg-primary-600 hover:text-white">
                            <i class="fas fa-eye"></i>
                        </button>
                    </div>
                    <div class="p-5 flex-1 flex flex-col">
                        <div class="text-xs text-primary-600 dark:text-primary-400 font-semibold uppercase tracking-wide mb-1">
                            ${storeData.categories.find(c => c.id === product.category)?.name || product.category}
                        </div>
                        <h3 class="font-bold text-lg mb-2 line-clamp-2 group-hover:text-primary-600 transition-colors">${product.name}</h3>
                        <p class="text-gray-600 dark:text-gray-400 text-sm mb-4 line-clamp-2 flex-1">${product.description}</p>
                        <div class="flex items-center justify-between mt-auto">
                            <span class="text-2xl font-bold text-gray-900 dark:text-white">$${product.price.toFixed(2)}</span>
                            <button onclick="addToCart(${product.id})" class="px-4 py-2 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-medium transition-all hover:scale-105 active:scale-95 flex items-center gap-2">
                                <i class="fas fa-plus"></i>
                                <span>Add</span>
                            </button>
                        </div>
                    </div>
                </div>
            `).join('');
        }

        function filterCategory(category) {
            currentCategory = category;
            updateUI();
        }

        function filterFunction(func) {
            currentFunction = currentFunction === func ? 'all' : func;
            updateUI();
        }

        function handleSearch(e) {
            const query = e.target.value.toLowerCase();
            const grid = document.getElementById('productsGrid');
            const noResults = document.getElementById('noResults');
            
            const filtered = storeData.products.filter(p => 
                p.name.toLowerCase().includes(query) || 
                p.description.toLowerCase().includes(query) ||
                p.function?.toLowerCase().includes(query)
            );
            
            if (filtered.length === 0) {
                grid.innerHTML = '';
                noResults.classList.remove('hidden');
                return;
            }
            
            noResults.classList.add('hidden');
            grid.innerHTML = filtered.map(product => `
                <div class="group bg-white dark:bg-gray-800 rounded-2xl shadow-sm hover:shadow-xl transition-all duration-300 overflow-hidden border border-gray-100 dark:border-gray-700 flex flex-col">
                    <div class="relative overflow-hidden aspect-square bg-gray-100 dark:bg-gray-700">
                        <img src="${product.image}" alt="${product.name}" class="w-full h-full object-cover transform group-hover:scale-110 transition-transform duration-500">
                        <button onclick="openProductModal(${product.id})" class="absolute bottom-3 right-3 w-10 h-10 bg-white dark:bg-gray-900 rounded-full shadow-lg flex items-center justify-center opacity-0 group-hover:opacity-100 transform translate-y-4 group-hover:translate-y-0 transition-all duration-300 hover:bg-primary-600 hover:text-white">
                            <i class="fas fa-eye"></i>
                        </button>
                    </div>
                    <div class="p-5 flex-1 flex flex-col">
                        <h3 class="font-bold text-lg mb-2 line-clamp-2">${product.name}</h3>
                        <p class="text-gray-600 dark:text-gray-400 text-sm mb-4 line-clamp-2 flex-1">${product.description}</p>
                        <div class="flex items-center justify-between mt-auto">
                            <span class="text-2xl font-bold text-gray-900 dark:text-white">$${product.price.toFixed(2)}</span>
                            <button onclick="addToCart(${product.id})" class="px-4 py-2 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-medium transition-all hover:scale-105">
                                <i class="fas fa-plus"></i>
                            </button>
                        </div>
                    </div>
                </div>
            `).join('');
        }

        // CART FUNCTIONS
        function addToCart(productId) {
            const product = storeData.products.find(p => p.id === productId);
            const existingItem = cart.find(item => item.id === productId);
            
            if (existingItem) {
                existingItem.quantity++;
            } else {
                cart.push({ ...product, quantity: 1 });
            }
            
            updateCartUI();
            showToast(`${product.name} added to cart`);
        }

        function removeFromCart(productId) {
            cart = cart.filter(item => item.id !== productId);
            updateCartUI();
        }

        function updateQuantity(productId, change) {
            const item = cart.find(item => item.id === productId);
            if (item) {
                item.quantity += change;
                if (item.quantity <= 0) {
                    removeFromCart(productId);
                } else {
                    updateCartUI();
                }
            }
        }

        function updateCartUI() {
            const badge = document.getElementById('cartBadge');
            const itemsContainer = document.getElementById('cartItems');
            const totalElement = document.getElementById('cartTotal');
            
            const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
            const totalPrice = cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
            
            if (totalItems > 0) {
                badge.textContent = totalItems;
                badge.classList.remove('hidden');
            } else {
                badge.classList.add('hidden');
            }
            
            totalElement.textContent = '$' + totalPrice.toFixed(2);
            
            if (cart.length === 0) {
                itemsContainer.innerHTML = `
                    <div class="text-center text-gray-500 dark:text-gray-400 mt-10">
                        <i class="fas fa-shopping-basket text-6xl mb-4 opacity-30"></i>
                        <p>Your cart is empty</p>
                    </div>
                `;
            } else {
                itemsContainer.innerHTML = cart.map(item => `
                    <div class="flex gap-4 bg-gray-50 dark:bg-gray-800 p-4 rounded-xl">
                        <img src="${item.image}" alt="${item.name}" class="w-20 h-20 object-cover rounded-lg">
                        <div class="flex-1">
                            <h4 class="font-semibold text-sm mb-1 line-clamp-1">${item.name}</h4>
                            <p class="text-primary-600 dark:text-primary-400 font-bold">$${item.price.toFixed(2)}</p>
                            <div class="flex items-center gap-3 mt-2">
                                <button onclick="updateQuantity(${item.id}, -1)" class="w-6 h-6 rounded-full bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 flex items-center justify-center text-xs">
                                    <i class="fas fa-minus"></i>
                                </button>
                                <span class="text-sm font-medium">${item.quantity}</span>
                                <button onclick="updateQuantity(${item.id}, 1)" class="w-6 h-6 rounded-full bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 flex items-center justify-center text-xs">
                                    <i class="fas fa-plus"></i>
                                </button>
                                <button onclick="removeFromCart(${item.id})" class="ml-auto text-red-500 hover:text-red-600 text-xs">
                                    <i class="fas fa-trash"></i>
                                </button>
                            </div>
                        </div>
                    </div>
                `).join('');
            }
        }

        function toggleCart() {
            const sidebar = document.getElementById('cartSidebar');
            const content = document.getElementById('cartContent');
            
            if (sidebar.classList.contains('hidden')) {
                sidebar.classList.remove('hidden');
                setTimeout(() => content.classList.remove('translate-x-full'), 10);
            } else {
                content.classList.add('translate-x-full');
                setTimeout(() => sidebar.classList.add('hidden'), 300);
            }
        }

        // CHECKOUT
        function checkout() {
            if (cart.length === 0) {
                showToast('Your cart is empty');
                return;
            }
            if (!storeData.currentUser) {
                showToast('Please sign in to checkout');
                openAuth();
                return;
            }
            
            const total = cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
            document.getElementById('checkoutTotal').textContent = '$' + total.toFixed(2);
            document.getElementById('checkoutModal').classList.remove('hidden');
            document.getElementById('checkoutModal').classList.add('flex');
        }

        function closeCheckout() {
            document.getElementById('checkoutModal').classList.add('hidden');
            document.getElementById('checkoutModal').classList.remove('flex');
        }

        function handlePaymentChange(e) {
            const cardElement = document.getElementById('cardElement');
            if (e.target.value === 'card') {
                cardElement.classList.remove('hidden');
                // Initialize Stripe Elements here if needed
            } else {
                cardElement.classList.add('hidden');
            }
        }

        function processOrder(e) {
            e.preventDefault();
            
            const paymentMethod = document.getElementById('paymentMethod').value;
            const total = cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
            
            const order = {
                id: 'ORD-' + Date.now(),
                customer: {
                    name: e.target.querySelector('input[type="text"]').value + ' ' + e.target.querySelectorAll('input[type="text"]')[1].value,
                    email: e.target.querySelector('input[type="email"]').value,
                    phone: e.target.querySelector('input[type="tel"]').value,
                    address: e.target.querySelector('textarea').value
                },
                items: [...cart],
                total: total,
                paymentMethod: paymentMethod,
                status: 'pending',
                date: new Date().toISOString()
            };
            
            storeData.orders.push(order);
            
            // Update stock
            cart.forEach(item => {
                const product = storeData.products.find(p => p.id === item.id);
                if (product) product.stock -= item.quantity;
            });
            
            saveData();
            cart = [];
            updateCartUI();
            closeCheckout();
            toggleCart();
            
            showToast('Order placed successfully!');
            
            // Send email notification (simulated)
            console.log('Email notification sent to:', storeData.settings.supportEmail);
        }

        // PRODUCT MODAL
        function openProductModal(productId) {
            const product = storeData.products.find(p => p.id === productId);
            const modal = document.getElementById('productModal');
            const content = document.getElementById('modalContent');
            
            content.innerHTML = `
                <div class="grid md:grid-cols-2 gap-6">
                    <div class="aspect-square bg-gray-100 dark:bg-gray-800 rounded-t-2xl md:rounded-l-2xl md:rounded-tr-none overflow-hidden">
                        <img src="${product.image}" alt="${product.name}" class="w-full h-full object-cover">
                    </div>
                    <div class="p-6 md:pl-0 md:pr-6 md:py-6">
                        <div class="text-sm text-primary-600 dark:text-primary-400 font-semibold uppercase tracking-wide mb-2">
                            ${storeData.categories.find(c => c.id === product.category)?.name}
                        </div>
                        <h2 class="text-3xl font-bold mb-2">${product.name}</h2>
                        ${product.function ? `<span class="inline-block bg-cyan-100 dark:bg-cyan-900/30 text-cyan-800 dark:text-cyan-300 text-sm px-3 py-1 rounded-full mb-4">${product.function}</span>` : ''}
                        <p class="text-3xl font-bold text-primary-600 dark:text-primary-400 mb-6">$${product.price.toFixed(2)}</p>
                        
                        <p class="text-gray-600 dark:text-gray-400 mb-6 leading-relaxed">${product.description}</p>
                        
                        <div class="mb-6">
                            <h3 class="font-semibold mb-3">Specifications:</h3>
                            <ul class="space-y-2">
                                ${product.specs.map(spec => `
                                    <li class="flex items-center text-sm text-gray-600 dark:text-gray-400">
                                        <i class="fas fa-check text-green-500 mr-2"></i>
                                        ${spec}
                                    </li>
                                `).join('')}
                            </ul>
                        </div>
                        
                        <div class="flex items-center gap-4 mb-6">
                            <div class="flex items-center gap-2 text-sm">
                                <span class="w-3 h-3 rounded-full ${product.stock > 10 ? 'bg-green-500' : 'bg-red-500'}"></span>
                                <span class="text-gray-600 dark:text-gray-400">${product.stock > 0 ? (product.stock > 10 ? 'In Stock' : `Only ${product.stock} left`) : 'Out of Stock'}</span>
                            </div>
                        </div>
                        
                        <button onclick="addToCart(${product.id}); closeProductModal();" class="w-full py-3 bg-primary-600 hover:bg-primary-700 text-white rounded-lg font-semibold transition-colors flex items-center justify-center gap-2" ${product.stock === 0 ? 'disabled' : ''}>
                            <i class="fas fa-shopping-cart"></i>
                            <span>${product.stock === 0 ? 'Out of Stock' : 'Add to Cart'}</span>
                        </button>
                    </div>
                </div>
            `;
            
            modal.classList.remove('hidden');
            document.body.style.overflow = 'hidden';
        }

        function closeProductModal() {
            document.getElementById('productModal').classList.add('hidden');
            document.body.style.overflow = '';
        }

        // CHAT WIDGET
        function toggleChat() {
            const chatWindow = document.getElementById('chatWindow');
            chatOpen = !chatOpen;
            
            if (chatOpen) {
                chatWindow.classList.remove('hidden');
                // Generate session ID if not exists
                if (!window.chatSessionId) {
                    window.chatSessionId = 'session-' + Date.now();
                    storeData.chats[window.chatSessionId] = {
                        customerName: storeData.currentUser?.name || 'Guest',
                        messages: [{
                            sender: 'bot',
                            text: `Hello! Welcome to ${storeData.settings.siteName}. How can I help you today?`,
                            time: new Date().toISOString(),
                            read: true
                        }]
                    };
                    saveData();
                }
            } else {
                chatWindow.classList.add('hidden');
            }
        }

        function sendMessage(e) {
            e.preventDefault();
            const input = document.getElementById('chatInput');
            const text = input.value.trim();
            if (!text) return;
            
            // Add user message
            const chat = storeData.chats[window.chatSessionId];
            chat.messages.push({
                sender: 'user',
                text,
                time: new Date().toISOString(),
                read: false
            });
            
            saveData();
            input.value = '';
            renderChatMessages();
            
            // Update admin panel if open
            if (!document.getElementById('adminPanel').classList.contains('hidden')) {
                updateChatSessions();
            }
            
            // Simulate auto-reply after delay
            setTimeout(() => {
                chat.messages.push({
                    sender: 'bot',
                    text: 'Thank you for your message. Our support team will respond shortly. For urgent inquiries, please email us at ' + storeData.settings.supportEmail,
                    time: new Date().toISOString(),
                    read: false
                });
                saveData();
                renderChatMessages();
                if (!document.getElementById('adminPanel').classList.contains('hidden')) {
                    updateChatSessions();
                }
            }, 2000);
        }

        function renderChatMessages() {
            const container = document.getElementById('chatMessages');
            const chat = storeData.chats[window.chatSessionId];
            
            container.innerHTML = chat.messages.map(m => `
                <div class="flex gap-3 ${m.sender === 'user' ? 'justify-end' : ''}">
                    ${m.sender !== 'user' ? `
                        <div class="w-8 h-8 bg-primary-600 rounded-full flex items-center justify-center text-white text-sm flex-shrink-0">
                            <i class="fas fa-${m.sender === 'bot' ? 'robot' : 'headset'}"></i>
                        </div>
                    ` : ''}
                    <div class="${m.sender === 'user' ? 'bg-primary-600 text-white' : 'bg-white dark:bg-gray-800'} p-3 rounded-lg ${m.sender === 'user' ? 'rounded-tr-none' : 'rounded-tl-none'} shadow-sm max-w-[80%]">
                        <p class="text-sm">${m.text}</p>
                        <span class="text-xs ${m.sender === 'user' ? 'text-primary-100' : 'text-gray-400'} mt-1 block">${new Date(m.time).toLocaleTimeString()}</span>
                    </div>
                    ${m.sender === 'user' ? `
                        <div class="w-8 h-8 bg-gray-300 dark:bg-gray-700 rounded-full flex items-center justify-center text-gray-600 dark:text-gray-300 text-sm flex-shrink-0">
                            <i class="fas fa-user"></i>
                        </div>
                    ` : ''}
                </div>
            `).join('');
            
            container.scrollTop = container.scrollHeight;
        }

        // THEME
        function toggleTheme() {
            if (document.documentElement.classList.contains('dark')) {
                document.documentElement.classList.remove('dark');
                localStorage.theme = 'light';
            } else {
                document.documentElement.classList.add('dark');
                localStorage.theme = 'dark';
            }
        }

        function loadTheme() {
            if (localStorage.theme === 'dark' || (!('theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
                document.documentElement.classList.add('dark');
            }
        }

        // UTILITIES
        function showToast(message) {
            const toast = document.getElementById('toast');
            document.getElementById('toastMessage').textContent = message;
            toast.classList.remove('translate-y-20', 'opacity-0');
            setTimeout(() => {
                toast.classList.add('translate-y-20', 'opacity-0');
            }, 3000);
        }

        function toggleMobileMenu() {
            // Implementation for mobile menu
        }

        // Close modals on escape
        document.addEventListener('keydown', (e) => {
            if (e.key === 'Escape') {
                closeProductModal();
                closeCheckout();
                closeAuth();
                if (chatOpen) toggleChat();
            }
        });
    </script>
</body>
</html>
