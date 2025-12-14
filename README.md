import 'package:flutter/material.dart';
import 'package:math_expressions/math_expressions.dart';
import 'dart:math';

void main() {
  runApp(CalculatorApp());
}

class CalculatorApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Scientific Calculator',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blueGrey,
        brightness: Brightness.dark, // Темная тема для лучшего вида
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: ScientificCalculator(),
    );
  }
}

class ScientificCalculator extends StatefulWidget {
  @override
  _ScientificCalculatorState createState() => _ScientificCalculatorState();
}

class _ScientificCalculatorState extends State<ScientificCalculator> {
  String _expression = '';
  String _result = '0';
  List<String> _history = []; // Список для хранения истории вычислений

  // Дополнительные кнопки научного режима
  static const List<String> scientificButtons = [
    'sin', 'cos', 'tan', 'ln', 'log', 'sqrt', '(', ')', '^'
  ];

  static const Color _buttonColor = Color(0xFF333333);
  static const Color _operatorColor = Color(0xFFFF9800);
  static const Color _scientificColor = Color(0xFF1976D2);
  static const Color _clearColor = Color(0xFFa5a5a5);

  // --- ЛОГИКА КАЛЬКУЛЯТОРА ---

  void _onButtonPressed(String buttonText) {
    setState(() {
      if (buttonText == 'C') {
        _expression = '';
        _result = '0';
      } else if (buttonText == 'DEL') {
        if (_expression.isNotEmpty) {
          _expression = _expression.substring(0, _expression.length - 1);
        }
      } else if (buttonText == '=') {
        _calculateResult();
      } else if (scientificButtons.contains(buttonText)) {
        // Добавление функций (например, sin() или sqrt())
        _expression += buttonText + '(';
      } else {
        _expression += buttonText;
      }
    });
  }

  void _calculateResult() {
    String finalExpression = _expression.replaceAll('x', '*');

    try {
      // Замена синтаксиса Dart на синтаксис, понятный пакету math_expressions
      finalExpression = finalExpression
          .replaceAll('ln', 'log')  // Math-expressions использует log для натурального логарифма
          .replaceAll('log', 'log10'); // Используем log10 для десятичного логарифма
      
      Parser p = Parser();
      Expression exp = p.parse(finalExpression);
      ContextModel cm = ContextModel();
      
      // Добавляем Pi и E для использования в выражениях
      cm.bindVariable(Variable('pi'), Number(pi));
      cm.bindVariable(Variable('e'), Number(e));

      double evalResult = exp.evaluate(EvaluationType.REAL, cm);

      // Форматирование результата
      String formattedResult = evalResult.toString();
      if (formattedResult.endsWith('.0')) {
        formattedResult = formattedResult.substring(0, formattedResult.length - 2);
      }
      
      // Добавление в историю только если выражение не пустое и не равно результату
      if (_expression.isNotEmpty && _expression != formattedResult) {
         _history.insert(0, '$_expression = $formattedResult');
      }
      
      _result = formattedResult;
      _expression = formattedResult; // Результат становится началом нового выражения
      
    } catch (e) {
      _result = 'Ошибка';
      _expression = '';
    }
  }

  // --- UI (ПОЛЬЗОВАТЕЛЬСКИЙ ИНТЕРФЕЙС) ---

  Widget _buildButton(String buttonText, Color color, {int flex = 1}) {
    return Expanded(
      flex: flex,
      child: Padding(
        padding: const EdgeInsets.all(6.0),
        child: MaterialButton(
          padding: EdgeInsets.all(20.0),
          color: color,
          shape: CircleBorder(),
          onPressed: () => _onButtonPressed(buttonText),
          child: Text(
            buttonText,
            style: TextStyle(
              fontSize: 22.0, // Уменьшаем размер для размещения науч. кнопок
              fontWeight: FontWeight.bold,
              color: (color == _clearColor) ? Colors.black : Colors.white,
            ),
          ),
        ),
      ),
    );
  }

  // Виджет для отображения истории
  Widget _buildHistoryDrawer() {
    return Drawer(
      child: Container(
        color: Colors.black87,
        child: Column(
          children: <Widget>[
            // Заголовок
            AppBar(
              title: Text("История вычислений"),
              automaticallyImplyLeading: false, 
              backgroundColor: Colors.blueGrey,
            ),
            // Список истории
            Expanded(
              child: ListView.builder(
                itemCount: _history.length,
                itemBuilder: (context, index) {
                  return ListTile(
                    title: Text(_history[index], style: TextStyle(color: Colors.white70, fontSize: 16)),
                    onTap: () {
                      // При нажатии на элемент истории, подставить его результат в выражение
                      String historyResult = _history[index].split('=').last.trim();
                      setState(() {
                        _expression = historyResult;
                        _result = historyResult;
                      });
                      Navigator.pop(context); // Закрыть боковое меню
                    },
                  );
                },
              ),
            ),
            // Кнопка очистки истории
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: ElevatedButton(
                onPressed: () => setState(() => _history.clear()),
                child: Text("Очистить историю", style: TextStyle(color: Colors.white)),
                style: ElevatedButton.styleFrom(
                  backgroundColor: _operatorColor,
                  minimumSize: Size(double.infinity, 50),
                ),
              ),
            )
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    // Используем Scaffold с ключом для управления боковым меню
    return Scaffold(
      backgroundColor: Colors.black,
      endDrawer: _buildHistoryDrawer(),
      body: SafeArea(
        child: Column(
          children: <Widget>[
            // --- ЭКРАН РЕЗУЛЬТАТОВ ---
            Builder(
              builder: (context) => Padding(
                padding: const EdgeInsets.only(top: 10.0, right: 10.0),
                child: Align(
                  alignment: Alignment.topRight,
                  child: IconButton(
                    icon: Icon(Icons.history, color: Colors.white70, size: 30),
                    onPressed: () => Scaffold.of(context).openEndDrawer(), // Открытие истории
                  ),
                ),
              ),
            ),
            Expanded(
              child: Container(
                padding: EdgeInsets.symmetric(horizontal: 20, vertical: 24),
                alignment: Alignment.bottomRight,
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.end,
                  crossAxisAlignment: CrossAxisAlignment.end,
                  children: [
                    Text(
                      _expression,
                      style: TextStyle(fontSize: 32.0, color: Colors.white70),
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                    ),
                    SizedBox(height: 10),
                    Text(
                      _result,
                      style: TextStyle(fontSize: 64.0, color: Colors.white),
                      maxLines: 1,
                      overflow: TextOverflow.ellipsis,
                    ),
                  ],
                ),
              ),
            ),
            
            // --- КЛАВИАТУРА ---
            Divider(height: 1, color: Colors.white24),
            
            // Добавляем ряд с научными функциями
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                _buildButton('sin', _scientificColor),
                _buildButton('cos', _scientificColor),
                _buildButton('tan', _scientificColor),
                _buildButton('^', _scientificColor),
              ],
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                _buildButton('ln', _scientificColor),
                _buildButton('log', _scientificColor),
                _buildButton('sqrt', _scientificColor),
                _buildButton('%', _operatorColor),
              ],
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                _buildButton('C', _clearColor),
                _buildButton('DEL', _clearColor),
                _buildButton('(', _scientificColor),
                _buildButton(')', _scientificColor),
              ],
            ),
            // Основная клавиатура
            Row(
              children: [
                Expanded(
                  flex: 3,
                  child: Column(
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                        children: <Widget>[
                          _buildButton('7', _buttonColor),
                          _buildButton('8', _buttonColor),
                          _buildButton('9', _buttonColor),
                        ],
                      ),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                        children: <Widget>[
                          _buildButton('4', _buttonColor),
                          _buildButton('5', _buttonColor),
                          _buildButton('6', _buttonColor),
                        ],
                      ),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                        children: <Widget>[
                          _buildButton('1', _buttonColor),
                          _buildButton('2', _buttonColor),
                          _buildButton('3', _buttonColor),
                        ],
                      ),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                        children: <Widget>[
                          _buildButton('0', _buttonColor, flex: 2), 
                          _buildButton('.', _buttonColor),
                          _buildButton('π', _scientificColor),
                        ],
                      ),
                    ],
                  ),
                ),
                Expanded(
                  flex: 1,
                  child: Column(
                    children: <Widget>[
                      _buildButton('/', _operatorColor),
                      _buildButton('x', _operatorColor),
                      _buildButton('-', _operatorColor),
                      _buildButton('+', _operatorColor),
                      _buildButton('=', _operatorColor),
                    ].expand((widget) => [widget, SizedBox(height: 1)]).toList()..removeLast(), // Добавляем небольшие отступы
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
