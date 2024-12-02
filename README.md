import 'package:flutter/material.dart';
import 'package:mqtt_client/mqtt_client.dart';

import 'mqtt_service.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'IOT Drop-Off System',
      theme: ThemeData(
        primaryColor: Colors.blueGrey[800],
        colorScheme: ColorScheme.fromSwatch().copyWith(secondary: Colors.tealAccent),
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: DoorControlPage(),
    );
  }
}

class DoorControlPage extends StatefulWidget {
  @override
  _DoorControlPageState createState() => _DoorControlPageState();
}

class _DoorControlPageState extends State<DoorControlPage> {
  final mqttService = MQTTService();
  String doorStatus = "Unknown";
  String connectionStatus = "Connecting...";
  bool objectDetected = false;

  @override
  void initState() {
    super.initState();
    _connectToMqtt();
  }

  Future<void> _connectToMqtt() async {
    await mqttService.connect();
    if (mqttService.isConnected) {
      setState(() {
        connectionStatus = "Connected";
      });
      mqttService.subscribe('home/door/status');
      mqttService.subscribe('home/door/notification');

      mqttService.client.updates!.listen((messages) {
        final recMess = messages[0].payload as MqttPublishMessage;
        final message = MqttPublishPayload.bytesToStringAsString(recMess.payload.message);
        final topic = messages[0].topic;

        setState(() {
          if (topic == 'home/door/status') {
            doorStatus = message;
          } else if (topic == 'home/door/notification' && message == "Object detected within 15 cm") {
            objectDetected = true;
            _showUnlockDialog();
          }
        });
      });
    } else {
      setState(() {
        connectionStatus = "Connection Failed";
      });
    }
  }

  void lockDoor() {
    mqttService.publish('home/door/control', 'lock');
  }

  void unlockDoor() {
    mqttService.publish('home/door/control', 'unlock');
  }

  void _showUnlockDialog() {
    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text("Object Detected!"),
          content: Text("An object is near the door. Do you want to unlock?"),
          actions: [
            TextButton(
              onPressed: () {
                unlockDoor();
                Navigator.of(context).pop();
              },
              child: Text("Unlock"),
            ),
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
              },
              child: Text("Cancel"),
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    final screenPadding = MediaQuery.of(context).size.width * 0.05;

    return Scaffold(
      appBar: AppBar(
        title: Text("IOT Drop-Off System"),
        centerTitle: true,
      ),
      body: SingleChildScrollView(
        padding: EdgeInsets.symmetric(horizontal: screenPadding, vertical: 20),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            _buildStatusCard("Connection Status", connectionStatus, Icons.wifi, Colors.blueGrey),
            SizedBox(height: 20),
            _buildStatusCard("Door Status", doorStatus, Icons.lock, Colors.orangeAccent),
            SizedBox(height: 40),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Expanded(
                  child: _buildControlButton("Lock Door", Icons.lock_outline, Colors.redAccent, lockDoor),
                ),
                SizedBox(width: 20),
                Expanded(
                  child: _buildControlButton("Unlock Door", Icons.lock_open, Colors.greenAccent, unlockDoor),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildStatusCard(String title, String status, IconData icon, Color color) {
    return Card(
      elevation: 4,
      color: Colors.white,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(15),
      ),
      child: ListTile(
        leading: Icon(icon, color: color, size: 36),
        title: Text(
          title,
          style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
        ),
        subtitle: Text(
          status,
          style: TextStyle(fontSize: 16, color: Colors.grey[600]),
        ),
      ),
    );
  }

  Widget _buildControlButton(String text, IconData icon, Color color, VoidCallback onPressed) {
    return ElevatedButton.icon(
      style: ElevatedButton.styleFrom(
        backgroundColor: color,
        padding: EdgeInsets.symmetric(vertical: 14),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(10)),
        elevation: 4,
      ),
      onPressed: onPressed,
      icon: Icon(icon, size: 28),
      label: Text(
        text,
        style: TextStyle(fontSize: 16),
      ),
    );
  }
}
