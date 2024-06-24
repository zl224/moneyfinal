import 'package:flutter/material.dart';
import 'dart:math';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '記帳App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        brightness: Brightness.light,
      ),
      darkTheme: ThemeData(
        brightness: Brightness.dark,
      ),
      initialRoute: '/',
      routes: {
        '/': (context) => LoginScreen(),
        '/signup': (context) => SignupScreen(),
        '/home': (context) => HomeScreen(),
      },
    );
  }
}

class User {
  String username;
  String password;

  User(this.username, this.password);
}

class UserDatabase {
  static final Map<String, User> _users = {};

  static bool isUsernameTaken(String username) {
    return _users.containsKey(username);
  }

  static bool validateUser(String username, String password) {
    if (_users.containsKey(username)) {
      return _users[username]!.password == password;
    }
    return false;
  }

  static void addUser(String username, String password) {
    _users[username] = User(username, password);
  }
}

class LoginScreen extends StatelessWidget {
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  void _login(BuildContext context) {
    String username = _usernameController.text;
    String password = _passwordController.text;

    if (UserDatabase.validateUser(username, password)) {
      Navigator.pushReplacementNamed(context, '/home');
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('帳號或密碼錯誤')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('登入')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _usernameController,
              decoration: InputDecoration(
                labelText: '帳號',
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () => _usernameController.clear(),
                ),
              ),
            ),
            TextField(
              controller: _passwordController,
              decoration: InputDecoration(
                labelText: '密碼',
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () => _passwordController.clear(),
                ),
              ),
              obscureText: true,
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () => _login(context),
              child: Text('登入'),
            ),
            TextButton(
              onPressed: () {
                Navigator.pushNamed(context, '/signup');
              },
              child: Text('註冊'),
            ),
          ],
        ),
      ),
    );
  }
}

class SignupScreen extends StatelessWidget {
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  void _signup(BuildContext context) {
    String username = _usernameController.text;
    String password = _passwordController.text;

    if (UserDatabase.isUsernameTaken(username)) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('帳號已被申請')),
      );
    } else {
      UserDatabase.addUser(username, password);
      Navigator.pushReplacementNamed(context, '/home');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('註冊')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _usernameController,
              decoration: InputDecoration(
                labelText: '帳號',
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () => _usernameController.clear(),
                ),
              ),
            ),
            TextField(
              controller: _passwordController,
              decoration: InputDecoration(
                labelText: '密碼',
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () => _passwordController.clear(),
                ),
              ),
              obscureText: true,
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () => _signup(context),
              child: Text('註冊'),
            ),
          ],
        ),
      ),
    );
  }
}

class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  List<Map<String, String>> incomeRecords = [];
  List<Map<String, String>> expenseRecords = [];

  void _logout(BuildContext context) {
    Navigator.pushReplacementNamed(context, '/');
  }

  void _addIncome(Map<String, String> record) {
    setState(() {
      incomeRecords.add(record);
    });
  }

  void _addExpense(Map<String, String> record) {
    setState(() {
      expenseRecords.add(record);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('主畫面'),
        actions: [
          IconButton(
            icon: Icon(Icons.logout),
            onPressed: () => _logout(context),
          ),
        ],
      ),
      body: Column(
        children: [
          Expanded(
            child: Column(
              children: [
                ElevatedButton(
                  onPressed: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => RecordIncomeScreen(onAddIncome: _addIncome),
                      ),
                    );
                  },
                  child: Text('記錄收入'),
                ),
                ElevatedButton(
                  onPressed: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => RecordExpenseScreen(onAddExpense: _addExpense),
                      ),
                    );
                  },
                  child: Text('記錄支出'),
                ),
                ElevatedButton(
                  onPressed: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(builder: (context) => TrendChartScreen(incomeRecords: incomeRecords, expenseRecords: expenseRecords)),
                    );
                  },
                  child: Text('收入支出趨勢圖'),
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView(
              children: [
                ...incomeRecords.map((record) => ListTile(
                      title: Text('收入: \$${record['amount']}'),
                      subtitle: Text(record['description']!),
                    )),
                ...expenseRecords.map((record) => ListTile(
                      title: Text('支出: \$${record['amount']}'),
                      subtitle: Text(record['description']!),
                    )),
              ],
            ),
          ),
        ],
      ),
      bottomNavigationBar: BottomNavigationBar(
        items: [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: '主畫面',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.pie_chart),
            label: '趨勢圖',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.settings),
            label: '設置',
          ),
        ],
        onTap: (index) {
          if (index == 1) {
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => TrendChartScreen(incomeRecords: incomeRecords, expenseRecords: expenseRecords)),
            );
          }
        },
      ),
    );
  }
}

class RecordIncomeScreen extends StatelessWidget {
  final Function(Map<String, String>) onAddIncome;
  final TextEditingController _amountController = TextEditingController();
  final TextEditingController _descriptionController = TextEditingController();

  RecordIncomeScreen({required this.onAddIncome});

  void _recordIncome(BuildContext context) {
    String amount = _amountController.text;
    String description = _descriptionController.text;

    if (amount.isNotEmpty && description.isNotEmpty) {
      onAddIncome({'amount': amount, 'description': description});
      Navigator.pop(context);
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('請填寫所有字段')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('記錄收入')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _amountController,
              decoration: InputDecoration(
                labelText: '金額',
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () => _amountController.clear(),
                ),
              ),
              keyboardType: TextInputType.number,
            ),
            TextField(
              controller: _descriptionController,
              decoration: InputDecoration(
                labelText: '描述',
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () => _descriptionController.clear(),
                ),
              ),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () => _recordIncome(context),
              child: Text('記錄'),
            ),
          ],
        ),
      ),
    );
  }
}

class RecordExpenseScreen extends StatelessWidget {
  final Function(Map<String, String>) onAddExpense;
  final TextEditingController _amountController = TextEditingController();
  final TextEditingController _descriptionController = TextEditingController();

  RecordExpenseScreen({required this.onAddExpense});

  void _recordExpense(BuildContext context) {
    String amount = _amountController.text;
    String description = _descriptionController.text;

    if (amount.isNotEmpty && description.isNotEmpty) {
      onAddExpense({'amount': amount, 'description': description});
      Navigator.pop(context);
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('請填寫所有字段')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('記錄支出')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _amountController,
              decoration: InputDecoration(
                labelText: '金額',
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () => _amountController.clear(),
                ),
              ),
              keyboardType: TextInputType.number,
            ),
            TextField(
              controller: _descriptionController,
              decoration: InputDecoration(
                labelText: '描述',
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () => _descriptionController.clear(),
                ),
              ),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () => _recordExpense(context),
              child: Text('記錄'),
            ),
          ],
        ),
      ),
    );
  }
}

class TrendChartScreen extends StatelessWidget {
  final List<Map<String, String>> incomeRecords;
  final List<Map<String, String>> expenseRecords;

  TrendChartScreen({required this.incomeRecords, required this.expenseRecords});

  @override
  Widget build(BuildContext context) {
    double totalIncome = incomeRecords.fold(0, (sum, item) => sum + double.parse(item['amount']!));
    double totalExpense = expenseRecords.fold(0, (sum, item) => sum + double.parse(item['amount']!));

    return Scaffold(
      appBar: AppBar(title: Text('收入支出趨勢圖')),
      body: Center(
        child: CustomPaint(
          size: Size(200, 200),
          painter: PieChartPainter(totalIncome: totalIncome, totalExpense: totalExpense),
        ),
      ),
    );
  }
}

class PieChartPainter extends CustomPainter {
  final double totalIncome;
  final double totalExpense;

  PieChartPainter({required this.totalIncome, required this.totalExpense});

  @override
  void paint(Canvas canvas, Size size) {
    double total = totalIncome + totalExpense;
    double incomeAngle = (totalIncome / total) * 2 * pi;
    double expenseAngle = (totalExpense / total) * 2 * pi;

    Paint paint = Paint()
      ..style = PaintingStyle.fill;

    // Draw income slice
    paint.color = Colors.green;
    canvas.drawArc(Rect.fromLTWH(0, 0, size.width, size.height), -pi / 2, incomeAngle, true, paint);

    // Draw expense slice
    paint.color = Colors.red;
    canvas.drawArc(Rect.fromLTWH(0, 0, size.width, size.height), -pi / 2 + incomeAngle, expenseAngle, true, paint);

    // Draw legend
    TextPainter incomePainter = TextPainter(
      text: TextSpan(
        text: '收入',
        style: TextStyle(color: Colors.green),
      ),
      textDirection: TextDirection.ltr,
    )..layout();

    TextPainter expensePainter = TextPainter(
      text: TextSpan(
        text: '支出',
        style: TextStyle(color: Colors.red),
      ),
      textDirection: TextDirection.ltr,
    )..layout();

    incomePainter.paint(canvas, Offset(size.width + 10, size.height / 2 - incomePainter.height / 2));
    expensePainter.paint(canvas, Offset(size.width + 10, size.height / 2 + expensePainter.height / 2));
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) {
    return true;
  }
}
