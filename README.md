import 'dart:io';

import 'package:flutter/material.dart';
import 'package:gps_based_attendence_system/screens/academic_performance.dart';
import 'package:gps_based_attendence_system/screens/announcements.dart';
import 'package:gps_based_attendence_system/screens/help_center.dart';
import 'package:gps_based_attendence_system/screens/student.dart';
import 'package:image_picker/image_picker.dart';
import 'package:path_provider/path_provider.dart';
import 'package:shared_preferences/shared_preferences.dart';

import 'login_screen.dart';

// Modern Premium Color Palette
const Color _primaryColor = Color(0xFF1E3A8A); // Deep Royal Blue
const Color _secondaryColor = Color(0xFF3B82F6); // Bright Blue
const Color _accentColor = Color(0xFF06B6D4); // Cyan
const Color _backgroundColor = Color(0xFFF3F4F6); // Very light grey
const Color _cardColor = Color(0xFFFFFFFF);
const Color _textColor = Color(0xFF111827); // Dark Slate
const Color _lightTextColor = Color(0xFF6B7280); // Cool Grey
const Color _white = Color(0xFFFFFFFF);

// Success & Danger Colors for IN/OUT
const Color _inColor = Color(0xFF10B981); // Emerald Green
const Color _outColor = Color(0xFFEF4444); // Red

class HomeScreen extends StatefulWidget {
  final String userId;
  const HomeScreen({super.key, required this.userId});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  final GlobalKey<ScaffoldState> _scaffoldKey = GlobalKey<ScaffoldState>();
  File? _profileImage;
  late ScrollController _scrollController;

  // State variable to toggle IN / OUT (Aap isay logic se change kar sakte hain)
  bool isInsideGeofence = true;

  @override
  void initState() {
    super.initState();
    _scrollController = ScrollController();
    _loadProfileImage();
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  Future<void> _loadProfileImage() async {
    final prefs = await SharedPreferences.getInstance();
    final imagePath = prefs.getString('profile_image_path');
    if (imagePath != null) {
      final file = File(imagePath);
      if (await file.exists()) {
        setState(() => _profileImage = file);
      }
    }
  }

  Future<void> _pickImage() async {
    final ImagePicker picker = ImagePicker();
    final XFile? image = await picker.pickImage(source: ImageSource.gallery);
    if (image == null) return;

    final appDir = await getApplicationDocumentsDirectory();
    final saved = await File(image.path).copy('${appDir.path}/profile.jpg');

    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('profile_image_path', saved.path);

    setState(() => _profileImage = saved);
  }

  ImageProvider _getProfileImageProvider() {
    if (_profileImage != null) return FileImage(_profileImage!);
    return const AssetImage('assets/profile.png');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      key: _scaffoldKey,
      drawer: _buildDrawer(context),
      backgroundColor: _backgroundColor,
      body: Column(
        children: [
          _buildCurvedHeader(),
          Expanded(
            child: SingleChildScrollView(
              controller: _scrollController,
              physics: const BouncingScrollPhysics(),
              child: Padding(
                padding: const EdgeInsets.symmetric(
                  horizontal: 16,
                  vertical: 10,
                ),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.stretch,
                  children: [
                    // IN / OUT Status Card
                    _buildInOutStatusCard(),

                    const SizedBox(height: 16),

                    // Quick Stats & Live Info
                    Row(
                      children: [
                        Expanded(child: _quickStatsCard()),
                        const SizedBox(width: 12),
                        Expanded(child: _liveStatusCard()),
                      ],
                    ),

                    const SizedBox(height: 50),

                    // Ultra Modern Center Button
                    Center(child: _buildUltraModernButton()),

                    const SizedBox(height: 50),

                    // Department Info
                    _geofenceInfoCard(),
                    const SizedBox(height: 30),
                  ],
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }

  // Modern Curved Header overlapping the body
  Widget _buildCurvedHeader() {
    return Container(
      padding: EdgeInsets.only(
        top: MediaQuery.of(context).padding.top + 10,
        left: 16,
        right: 16,
        bottom: 30,
      ),
      decoration: const BoxDecoration(
        gradient: LinearGradient(
          colors: [_primaryColor, _secondaryColor],
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
        ),
        borderRadius: BorderRadius.only(
          bottomLeft: Radius.circular(30),
          bottomRight: Radius.circular(30),
        ),
        boxShadow: [
          BoxShadow(
            color: Color.fromRGBO(30, 58, 138, 0.4),
            blurRadius: 15,
            offset: Offset(0, 5),
          ),
        ],
      ),
      child: Column(
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              GestureDetector(
                onTap: () => _scaffoldKey.currentState?.openDrawer(),
                child: Container(
                  padding: const EdgeInsets.all(8),
                  decoration: BoxDecoration(
                    color: Colors.white.withOpacity(0.2),
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: const Icon(
                    Icons.grid_view_rounded,
                    color: _white,
                    size: 24,
                  ),
                ),
              ),
              const Text(
                'Dashboard',
                style: TextStyle(
                  color: _white,
                  fontSize: 20,
                  fontWeight: FontWeight.w700,
                  letterSpacing: 1.0,
                ),
              ),
              GestureDetector(
                onTap: _pickImage,
                child: Container(
                  decoration: BoxDecoration(
                    shape: BoxShape.circle,
                    border: Border.all(color: _white, width: 2),
                    boxShadow: [
                      BoxShadow(
                        color: Colors.black.withOpacity(0.2),
                        blurRadius: 8,
                      ),
                    ],
                  ),
                  child: CircleAvatar(
                    radius: 20,
                    backgroundColor: Colors.white24,
                    backgroundImage: _getProfileImageProvider(),
                  ),
                ),
              ),
            ],
          ),
          const SizedBox(height: 25),
          Row(
            children: [
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Welcome back,',
                      style: TextStyle(
                        color: Colors.white70,
                        fontSize: 16,
                        fontWeight: FontWeight.w500,
                      ),
                    ),
                    const SizedBox(height: 4),
                    Text(
                      widget.userId,
                      style: const TextStyle(
                        fontSize: 28,
                        fontWeight: FontWeight.bold,
                        color: _white,
                        fontFamily: 'Roboto',
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }

  // The attractive IN/OUT badge card
  Widget _buildInOutStatusCard() {
    return Container(
      margin: const EdgeInsets.only(top: 10),
      padding: const EdgeInsets.all(20),
      decoration: BoxDecoration(
        color: _cardColor,
        borderRadius: BorderRadius.circular(24),
        border: Border.all(color: Colors.white, width: 2),
        boxShadow: [
          BoxShadow(
            color: (isInsideGeofence ? _inColor : _outColor).withOpacity(0.15),
            blurRadius: 20,
            spreadRadius: 5,
            offset: const Offset(0, 8),
          ),
        ],
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                'Geofence Status',
                style: TextStyle(
                  color: _lightTextColor,
                  fontSize: 14,
                  fontWeight: FontWeight.w600,
                  letterSpacing: 0.5,
                ),
              ),
              const SizedBox(height: 8),
              Row(
                children: [
                  Icon(
                    isInsideGeofence
                        ? Icons.my_location_rounded
                        : Icons.location_off_rounded,
                    color: isInsideGeofence ? _inColor : _outColor,
                    size: 24,
                  ),
                  const SizedBox(width: 8),
                  Text(
                    isInsideGeofence ? 'Inside Campus' : 'Outside Campus',
                    style: TextStyle(
                      color: _textColor,
                      fontWeight: FontWeight.w800,
                      fontSize: 18,
                    ),
                  ),
                ],
              ),
            ],
          ),
          // The prominent IN / OUT Badge
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
            decoration: BoxDecoration(
              color: (isInsideGeofence ? _inColor : _outColor).withOpacity(0.1),
              borderRadius: BorderRadius.circular(20),
              border: Border.all(
                color: (isInsideGeofence ? _inColor : _outColor).withOpacity(
                  0.5,
                ),
                width: 2,
              ),
            ),
            child: Text(
              isInsideGeofence ? 'IN' : 'OUT',
              style: TextStyle(
                color: isInsideGeofence ? _inColor : _outColor,
                fontSize: 24,
                fontWeight: FontWeight.w900,
                letterSpacing: 2.0,
              ),
            ),
          ),
        ],
      ),
    );
  }

  // The Main Hero Button (Ultra Attractive)
  Widget _buildUltraModernButton() {
    return Column(
      children: [
        Stack(
          alignment: Alignment.center,
          children: [
            // Outer glowing ring
            Container(
              height: 180,
              width: 180,
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                color: _secondaryColor.withOpacity(0.1),
              ),
            ),
            // Middle ring
            Container(
              height: 150,
              width: 150,
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                color: _secondaryColor.withOpacity(0.2),
              ),
            ),
            // The actual button
            Container(
              height: 120,
              width: 120,
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                gradient: const LinearGradient(
                  colors: [_accentColor, _primaryColor],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
                boxShadow: [
                  BoxShadow(
                    color: _primaryColor.withOpacity(0.5),
                    blurRadius: 25,
                    spreadRadius: 5,
                    offset: const Offset(0, 10),
                  ),
                ],
                border: Border.all(
                  color: Colors.white.withOpacity(0.4),
                  width: 3,
                ),
              ),
              child: Material(
                color: Colors.transparent,
                child: InkWell(
                  customBorder: const CircleBorder(),
                  splashColor: Colors.white.withOpacity(0.4),
                  highlightColor: Colors.white.withOpacity(0.1),
                  onTap: () {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Processing Attendance via GPS...'),
                        backgroundColor: _primaryColor,
                        behavior: SnackBarBehavior.floating,
                      ),
                    );
                  },
                  child: const Center(
                    child: Icon(
                      Icons
                          .fingerprint_rounded, // Fingerprint implies attendance marking
                      color: Colors.white,
                      size: 60,
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
        const SizedBox(height: 24),
        const Text(
          'Mark Attendance',
          style: TextStyle(
            fontSize: 24,
            fontWeight: FontWeight.w900,
            color: _primaryColor,
            letterSpacing: 0.5,
          ),
        ),
        const SizedBox(height: 8),
        Container(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 6),
          decoration: BoxDecoration(
            color: _white,
            borderRadius: BorderRadius.circular(20),
            border: Border.all(color: Colors.grey.shade300),
          ),
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(Icons.gps_fixed_rounded, size: 16, color: _secondaryColor),
              const SizedBox(width: 6),
              const Text(
                'GPS Location Active',
                style: TextStyle(
                  fontSize: 13,
                  color: _textColor,
                  fontWeight: FontWeight.w600,
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }

  Widget _quickStatsCard() {
    return _glassCard(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Container(
            padding: const EdgeInsets.all(10),
            decoration: BoxDecoration(
              color: const Color.fromRGBO(30, 58, 138, 0.1),
              borderRadius: BorderRadius.circular(12),
            ),
            child: const Icon(
              Icons.analytics_rounded,
              color: _primaryColor,
              size: 24,
            ),
          ),
          const SizedBox(height: 14),
          const Text(
            "Today's Status",
            style: TextStyle(
              color: _lightTextColor,
              fontWeight: FontWeight.w600,
            ),
          ),
          const SizedBox(height: 6),
          const Text(
            'Pending',
            style: TextStyle(
              color: Colors.orangeAccent,
              fontWeight: FontWeight.w900,
              fontSize: 20,
            ),
          ),
        ],
      ),
    );
  }

  Widget _liveStatusCard() {
    return _glassCard(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Container(
            padding: const EdgeInsets.all(10),
            decoration: BoxDecoration(
              color: const Color.fromRGBO(6, 182, 212, 0.1),
              borderRadius: BorderRadius.circular(12),
            ),
            child: const Icon(
              Icons.wifi_tethering_rounded,
              color: _accentColor,
              size: 24,
            ),
          ),
          const SizedBox(height: 14),
          const Text(
            "System Status",
            style: TextStyle(
              color: _lightTextColor,
              fontWeight: FontWeight.w600,
            ),
          ),
          const SizedBox(height: 6),
          Row(
            children: [
              Container(
                height: 10,
                width: 10,
                decoration: const BoxDecoration(
                  color: _inColor,
                  shape: BoxShape.circle,
                ),
              ),
              const SizedBox(width: 6),
              const Text(
                'Online',
                style: TextStyle(
                  color: _textColor,
                  fontWeight: FontWeight.w900,
                  fontSize: 20,
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }

  Widget _geofenceInfoCard() {
    return _glassCard(
      child: Row(
        children: [
          Container(
            padding: const EdgeInsets.all(14),
            decoration: BoxDecoration(
              gradient: const LinearGradient(
                colors: [_secondaryColor, _primaryColor],
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
              ),
              borderRadius: BorderRadius.circular(16),
            ),
            child: const Icon(Icons.apartment_rounded, color: _white, size: 30),
          ),
          const SizedBox(width: 16),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: const [
                Text(
                  'Computer Science Dept',
                  style: TextStyle(
                    color: _textColor,
                    fontWeight: FontWeight.w800,
                    fontSize: 16,
                  ),
                ),
                SizedBox(height: 6),
                Text(
                  'Class Assigned: None',
                  style: TextStyle(
                    color: _lightTextColor,
                    fontSize: 14,
                    fontWeight: FontWeight.w500,
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _glassCard({required Widget child}) {
    return Container(
      padding: const EdgeInsets.all(18),
      decoration: BoxDecoration(
        color: _cardColor,
        borderRadius: BorderRadius.circular(20),
        border: Border.all(color: Colors.white, width: 2),
        boxShadow: const [
          BoxShadow(
            color: Color.fromRGBO(0, 0, 0, 0.03),
            blurRadius: 15,
            spreadRadius: 2,
            offset: Offset(0, 5),
          ),
        ],
      ),
      child: child,
    );
  }

  Widget _buildDrawer(BuildContext context) {
    return Drawer(
      backgroundColor: _primaryColor,
      child: Column(
        children: [
          DrawerHeader(
            decoration: const BoxDecoration(
              gradient: LinearGradient(
                colors: [_primaryColor, Color.fromARGB(255, 20, 20, 20)],
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
              ),
            ),
            child: Row(
              children: [
                Container(
                  padding: const EdgeInsets.all(3),
                  decoration: const BoxDecoration(
                    color: _white,
                    shape: BoxShape.circle,
                  ),
                  child: CircleAvatar(
                    radius: 35,
                    backgroundColor: _backgroundColor,
                    backgroundImage: _getProfileImageProvider(),
                  ),
                ),
                const SizedBox(width: 14),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text(
                        widget.userId,
                        style: const TextStyle(
                          color: _white,
                          fontWeight: FontWeight.w900,
                          fontSize: 22,
                        ),
                      ),
                      const SizedBox(height: 6),
                      Container(
                        padding: const EdgeInsets.symmetric(
                          horizontal: 8,
                          vertical: 4,
                        ),
                        decoration: BoxDecoration(
                          color: Colors.black26,
                          borderRadius: BorderRadius.circular(10),
                        ),
                        child: const Text(
                          'Student',
                          style: TextStyle(color: Colors.white, fontSize: 12),
                        ),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView(
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
              children: [
                _drawerItem(
                  Icons.home_rounded,
                  'Dashboard',
                  null,
                  onTap: () {
                    Navigator.pop(context);
                    setState(() {});
                  },
                ),
                _drawerItem(
                  Icons.person_rounded,
                  'My Profile',
                  const StudentPage(),
                ),
                _drawerItem(
                  Icons.campaign_rounded,
                  'Announcements',
                  const AnnouncementPage(),
                ),
                _drawerItem(
                  Icons.leaderboard_rounded,
                  'Academic Record',
                  const AcademicPage(),
                ),
                _drawerItem(
                  Icons.support_agent_rounded,
                  'Help Center',
                  const HelpCenterPage(),
                ),
              ],
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(20.0),
            child: ElevatedButton.icon(
              onPressed: () {
                Navigator.pop(context);
                Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(builder: (_) => const LoginScreen()),
                );
              },
              icon: const Icon(Icons.power_settings_new_rounded),
              label: const Text(
                'Sign Out',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.white.withOpacity(0.15),
                foregroundColor: _white,
                elevation: 0,
                minimumSize: const Size(double.infinity, 55),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(16),
                ),
              ),
            ),
          ),
          const SizedBox(height: 10),
        ],
      ),
    );
  }

  Widget _drawerItem(
    IconData icon,
    String title,
    Widget? page, {
    VoidCallback? onTap,
  }) {
    return Container(
      margin: const EdgeInsets.only(bottom: 6),
      child: ListTile(
        onTap:
            onTap ??
            () {
              Navigator.pop(context);
              if (page != null) {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) => page),
                );
              }
            },
        leading: Icon(
          icon,
          color: const Color.fromARGB(255, 29, 4, 105).withOpacity(0.9),
          size: 26,
        ),
        title: Text(
          title,
          style: const TextStyle(
            color: _white,
            fontSize: 16,
            fontWeight: FontWeight.w600,
            letterSpacing: 0.5,
          ),
        ),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(14)),
        hoverColor: Colors.white10,
        splashColor: Colors.white24,
      ),
    );
  }
}

// Dummy classes taake aapka code bina error ke chalay
class StudentPages extends StatelessWidget {
  const StudentPages({super.key});
  @override
  Widget build(BuildContext context) => const Scaffold(body: StudentPage());
}

class AnnouncementPage extends StatelessWidget {
  const AnnouncementPage({super.key});
  @override
  Widget build(BuildContext context) =>
      const Scaffold(body: AnnouncementsListScreen());
}

class HelpCenterPage extends StatelessWidget {
  const HelpCenterPage({super.key});
  @override
  Widget build(BuildContext context) =>
      const Scaffold(body: HelpCenterScreen());
}

class AcademicPage extends StatelessWidget {
  const AcademicPage({super.key});
  @override
  Widget build(BuildContext context) =>
      const Scaffold(body: AcademicHomePage());
}

