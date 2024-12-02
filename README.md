import 'package:flutter/material.dart';

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
  String doorStatus = "Unknown";
  String connectionStatus = "Not Connected";

  void lockDoor() {
    setState(() {
      doorStatus = "Locked";
    });
  }

  void unlockDoor() {
    setState(() {
      doorStatus = "Unlocked";
    });
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
