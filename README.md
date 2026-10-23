name: Build ReelMind APK

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'

      - name: Generate Flutter App & Build APK
        run: |
          mkdir build_app
          cd build_app
          flutter create --platforms=android --org com.reelmind --project-name reelmind .
          cat << 'EOF' > lib/main.dart
          import 'package:flutter/material.dart';

          void main() => runApp(const ReelMindApp());

          class ReelMindApp extends StatelessWidget {
            const ReelMindApp({super.key});
            @override
            Widget build(BuildContext context) {
              return MaterialApp(
                title: 'ReelMind AI',
                debugShowCheckedModeBanner: false,
                theme: ThemeData.dark().copyWith(
                  scaffoldBackgroundColor: const Color(0xFF0D0D12),
                  primaryColor: const Color(0xFF7C4DFF),
                  colorScheme: const ColorScheme.dark(
                    primary: Color(0xFF7C4DFF),
                    secondary: Color(0xFF00E5FF),
                    surface: Color(0xFF1A1A24),
                  ),
                ),
                home: const ReelMindHome(),
              );
            }
          }

          class ReelMindHome extends StatefulWidget {
            const ReelMindHome({super.key});
            @override
            State<ReelMindHome> createState() => _ReelMindHomeState();
          }

          class _ReelMindHomeState extends State<ReelMindHome> {
            int _tab = 1;
            final TextEditingController _prompt = TextEditingController();
            String? _img1, _vid1, _aud1;
            String _ratio = '9:16';
            String _duration = '5s';
            final List<Map<String, String>> _creations = [
              {
                'prompt': 'Animate @image1 character dancing to @audio1 with camera motion from @video1',
                'meta': '9:16 • 5s • @image1 @video1 @audio1'
              }
            ];

            void _addTag(String tag) {
              _prompt.text = '${_prompt.text} $tag ';
            }

            void _generate() {
              if (_prompt.text.trim().isEmpty) return;
              setState(() {
                _creations.insert(0, {
                  'prompt': _prompt.text.trim(),
                  'meta': '$_ratio • $_duration'
                });
                _tab = 2;
              });
            }

            @override
            Widget build(BuildContext context) {
              return Scaffold(
                appBar: AppBar(
                  backgroundColor: const Color(0xFF14141D),
                  title: const Row(
                    children: [
                      Icon(Icons.auto_awesome, color: Color(0xFF00E5FF)),
                      SizedBox(width: 8),
                      Text('ReelMind AI Studio', style: TextStyle(fontWeight: FontWeight.bold)),
                    ],
                  ),
                ),
                body: _tab == 0 ? _buildHome() : _tab == 1 ? _buildStudio() : _buildHistory(),
                bottomNavigationBar: BottomNavigationBar(
                  backgroundColor: const Color(0xFF14141D),
                  selectedItemColor: const Color(0xFF7C4DFF),
                  unselectedItemColor: Colors.grey,
                  currentIndex: _tab,
                  onTap: (i) => setState(() => _tab = i),
                  items: const [
                    BottomNavigationBarItem(icon: Icon(Icons.explore), label: 'Templates'),
                    BottomNavigationBarItem(icon: Icon(Icons.movie_filter), label: 'Studio'),
                    BottomNavigationBarItem(icon: Icon(Icons.video_library), label: 'Creations'),
                  ],
                ),
              );
            }

            Widget _buildHome() {
              final templates = ['Cyberpunk Action', 'Product Ad', 'Dance Sync', 'AI Storyteller'];
              return GridView.builder(
                padding: const EdgeInsets.all(16),
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 2, mainAxisSpacing: 12, crossAxisSpacing: 12, childAspectRatio: 0.85,
                ),
                itemCount: templates.length,
                itemBuilder: (ctx, i) => InkWell(
                  onTap: () => setState(() => _tab = 1),
                  child: Container(
                    padding: const EdgeInsets.all(12),
                    decoration: BoxDecoration(
                      color: const Color(0xFF1A1A24),
                      borderRadius: BorderRadius.circular(16),
                      border: Border.all(color: const Color(0xFF7C4DFF)),
                    ),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.end,
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        const Expanded(child: Center(child: Icon(Icons.play_circle_fill, size: 48, color: Color(0xFF00E5FF)))),
                        Text(templates[i], style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
                        const Text('Tap to use template', style: TextStyle(color: Colors.grey, fontSize: 12)),
                      ],
                    ),
                  ),
                ),
              );
            }

            Widget _buildStudio() {
              return ListView(
                padding: const EdgeInsets.all(16),
                children: [
                  const Text('1. Upload References (@image1, @video1, @audio1)', style: TextStyle(fontWeight: FontWeight.bold)),
                  const SizedBox(height: 10),
                  Row(
                    children: [
                      _refBox('@image1', _img1 ?? 'Select Image', Icons.image, _img1 != null, () => setState(() => _img1 = 'photo.png')),
                      const SizedBox(width: 8),
                      _refBox('@video1', _vid1 ?? 'Select Video', Icons.videocam, _vid1 != null, () => setState(() => _vid1 = 'motion.mp4')),
                      const SizedBox(width: 8),
                      _refBox('@audio1', _aud1 ?? 'Select Audio', Icons.audiotrack, _aud1 != null, () => setState(() => _aud1 = 'beat.mp3')),
                    ],
                  ),
                  const SizedBox(height: 20),
                  const Text('2. Tap to Insert Reference Tags in Prompt', style: TextStyle(fontWeight: FontWeight.bold)),
                  const SizedBox(height: 8),
                  Row(
                    children: ['@image1', '@video1', '@audio1'].map((t) => Padding(
                      padding: const EdgeInsets.only(right: 8),
                      child: ActionChip(label: Text('+ $t'), onPressed: () => _addTag(t)),
                    )).toList(),
                  ),
                  const SizedBox(height: 10),
                  TextField(
                    controller: _prompt,
                    maxLines: 4,
                    decoration: InputDecoration(
                      hintText: 'Combine @image1 character with @video1 motion and @audio1 sound...',
                      filled: true,
                      fillColor: const Color(0xFF1A1A24),
                      border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
                    ),
                  ),
                  const SizedBox(height: 20),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      DropdownButton<String>(
                        value: _ratio,
                        items: ['9:16', '16:9', '1:1'].map((e) => DropdownMenuItem(value: e, child: Text('Ratio: $e'))).toList(),
                        onChanged: (v) => setState(() => _ratio = v!),
                      ),
                      DropdownButton<String>(
                        value: _duration,
                        items: ['5s', '10s'].map((e) => DropdownMenuItem(value: e, child: Text('Length: $e'))).toList(),
                        onChanged: (v) => setState(() => _duration = v!),
                      ),
                    ],
                  ),
                  const SizedBox(height: 20),
                  SizedBox(
                    height: 52,
                    child: ElevatedButton.icon(
                      style: ElevatedButton.styleFrom(backgroundColor: const Color(0xFF7C4DFF)),
                      onPressed: _generate,
                      icon: const Icon(Icons.bolt, color: Colors.white),
                      label: const Text('Generate AI Video', style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold, fontSize: 16)),
                    ),
                  ),
                ],
              );
            }

            Widget _refBox(String tag, String sub, IconData icon, bool active, VoidCallback onTap) {
              return Expanded(
                child: InkWell(
                  onTap: onTap,
                  child: Container(
                    height: 95,
                    padding: const EdgeInsets.all(8),
                    decoration: BoxDecoration(
                      color: const Color(0xFF1A1A24),
                      borderRadius: BorderRadius.circular(12),
                      border: Border.all(color: active ? const Color(0xFF00E5FF) : Colors.white24),
                    ),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Icon(icon, color: active ? const Color(0xFF00E5FF) : Colors.grey),
                        Text(tag, style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 12)),
                        Text(sub, overflow: TextOverflow.ellipsis, style: const TextStyle(fontSize: 10, color: Colors.grey)),
                      ],
                    ),
                  ),
                ),
              );
            }

            Widget _buildHistory() {
              return ListView.builder(
                padding: const EdgeInsets.all(16),
                itemCount: _creations.length,
                itemBuilder: (ctx, i) => Card(
                  color: const Color(0xFF1A1A24),
                  margin: const EdgeInsets.only(bottom: 12),
                  child: ListTile(
                    leading: const Icon(Icons.play_circle_fill, size: 40, color: Color(0xFF00E5FF)),
                    title: Text(_creations[i]['prompt']!),
                    subtitle: Text(_creations[i]['meta']!),
                  ),
                ),
              );
            }
          }
          EOF
          flutter build apk --release

      - name: Upload Ready-to-Install APK
        uses: actions/upload-artifact@v4
        with:
          name: ReelMind-Android-APK
          path: build_app/build/app/outputs/flutter-apk/app-release.apk
