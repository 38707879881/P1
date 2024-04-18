import 'dart:convert';
import 'dart:io';

import 'package:device_preview/device_preview.dart';
import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';
import 'package:path_provider/path_provider.dart';

void main() {
  runApp(
    DevicePreview(
      enabled: true,
      builder: (context) => MainApp(),
    ),
  );
}

class MainApp extends StatelessWidget {
  const MainApp({Key? key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Lista',
      home: PrincipalView(),
    );
  }
}

class PrincipalView extends StatefulWidget {
  @override
  State<PrincipalView> createState() => _PrincipalViewState();
}

class _PrincipalViewState extends State<PrincipalView> {
  
  final _toDoController = TextEditingController();

  List _toDoList = [];
  late Map<String, dynamic> _lastRemoved;
  late int _lastRemovedPos;

  @override
  void initState() {
    // TODO: implement initState
    super.initState();

    _readData().then((data){
      setState(() {
        _toDoList = json.decode(data!);
      });
    });
  }

  void _addToDo(){
      setState(() {
        Map<String, dynamic> newToDo = Map();
      newToDo["title"] = _toDoController.text;
      _toDoController.text = "";
      newToDo["ok"] = false;
      _toDoList.add(newToDo);
      _saveData();
      });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Lista de Tarefas P1"),
        backgroundColor: Colors.blueAccent,
        centerTitle: true,
      ),
      body: Column(
        children: <Widget>[
          Container(
            padding: EdgeInsets.fromLTRB(17.0, 1.0, 7.0, 1.0),
            child: Row(
              children: <Widget>[
                Expanded(
                  child: TextField(
                    controller: _toDoController,
                    decoration: InputDecoration(
                      labelText: "Nova Tarefa",
                      labelStyle: TextStyle(color: Colors.black),
                    ),
                  ),
                ),
                ElevatedButton(
                  style: ButtonStyle(
                    backgroundColor:
                        MaterialStateProperty.all(Colors.blueAccent),
                    foregroundColor: MaterialStateProperty.all(Colors.white),
                  ),
                  child: Text("ADD"),
                  onPressed: _addToDo,
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              padding: EdgeInsets.only(top: 10.0),
              itemCount: _toDoList.length,
              itemBuilder: buildItem
            ),
          ),
        ],
      ),
    );
  }

  Widget buildItem(BuildContext context, int index) {
    return Dismissible(
      key: Key(DateTime.now().microsecondsSinceEpoch.toString()),
      background: Container(
        color: Colors.red,
        child: Align(
          alignment: Alignment(-0.9, 0.0),
          child: Icon(Icons.delete, color: Colors.white,),
        ),
      ),
      direction: DismissDirection.startToEnd,
      child:CheckboxListTile(
      title: Text(_toDoList[index]["title"]),
      value: _toDoList[index]["ok"],
      onChanged: (value) {
        setState(() {
          _toDoList[index]["ok"] = value;
          _saveData();
        });
      },
      secondary: CircleAvatar(
        child: Icon(_toDoList[index]["ok"] ? Icons.check : Icons.error),
      ),
    ),
    onDismissed: (direction) {
      setState(() {
        _lastRemoved = Map.from(_toDoList[index]);
      _lastRemovedPos = index;
      _toDoList.removeAt(index);

      _saveData();

      final snack = SnackBar(
        content: Text("Tarefa \"${_lastRemoved["title"]}\" Removida! "),
        action: SnackBarAction(label: "Desfazer",
        onPressed: () {
          setState(() {
            _toDoList.insert(_lastRemovedPos, _lastRemoved);
            _saveData();
          });
        }),
        duration: Duration(seconds: 2),
      );
      ScaffoldMessenger.of(context).showSnackBar(snack);
    });
  },
  );
  }


  Future<File> _getFile() async {
    final directory = await getApplicationDocumentsDirectory();
    return File("${directory.path}/data.json");
  }

  Future<File> _saveData() async {
    String data = json.encode(_toDoList);
    final file = await _getFile();
    return file.writeAsString(data);
  }

  Future<String?> _readData() async {
    try {
      final file = await _getFile();
      return file.readAsString();
    } catch (e) {
      return null;
    }
  }
}
