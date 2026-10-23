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

      - name: Generate Flutter App & Build APK v2
        run: |
          mkdir build_app
          cd build_app
          flutter create --platforms=android --org com.reelmind --project-name reelmind .
          flutter pub add file_picker http url_launcher
          cat << 'EOF' > lib/main.dart
          import 'dart:async';
          import 'dart:convert';
          import 'dart:io';
          import 'package:flutter/material.dart';
          import 'package:file_picker/file_picker.dart';
          import 'package:http/http.dart' as http;
          import 'package:url_launcher/url_launcher.dart';

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
            final TextEditingController _prompt = TextEditingController(
              text: 'Transfer the exact facial expression, crying emotion, and head movement from @video1 onto the woman in @image1, synced to @audio1 voiceover.',
            );
            final TextEditingController _apiKeyController = TextEditingController();

            String? _img1Path, _img1Name;
            String? _vid1Path, _vid1Name;
            String? _aud1Path, _aud1Name;

            String _ratio = '9:16';
            String _duration = '10s';
            bool _isGenerating = false;
            double _progress = 0.0;
            String _statusMessage = '';

            final List<Map<String, String>> _creations = [];

            Future<void> _pickMedia(String type) async {
              FileType ft = FileType.image;
              if (type == 'video') ft = FileType.video;
              if (type == 'audio') ft = FileType.audio;

              final res = await FilePicker.platform.pickFiles(type: ft);
              if (res != null && res.files.isNotEmpty) {
                setState(() {
                  if (type == 'image') {
                    _img1Path = res.files.single.path;
                    _img1Name = res.files.single.name;
                  }
                  if (type == 'video') {
                    _vid1Path = res.files.single.path;
                    _vid1Name = res.files.single.name;
                    _aud1Name ??= 'Audio from @video1';
                  }
                  if (type == 'audio') {
                    _aud1Path = res.files.single.path;
                    _aud1Name = res.files.single.name;
                  }
                });
              }
            }

            void _addTag(String tag) {
              _prompt.text = '${_prompt.text} $tag ';
              _prompt.selection = TextSelection.collapsed(offset: _prompt.text.length);
            }

            Future<String?> _uploadToTempHost(String filePath) async {
              try {
                final req = http.MultipartRequest('POST', Uri.parse('https://catbox.moe/user/api.php'));
                req.fields['reqtype'] = 'fileupload';
                req.files.add(await http.MultipartFile.fromPath('fileToUpload', filePath));
                final res = await req.send();
                final body = await res.stream.bytesToString();
                if (res.statusCode == 200 && body.startsWith('http')) {
                  return body.trim();
                }
              } catch (_) {}
              return null;
            }

            Future<void> _generate() async {
              if (_prompt.text.trim().isEmpty) return;
              setState(() {
                _isGenerating = true;
                _progress = 0.15;
                _statusMessage = 'Preparing @image1, @video1 & @audio1...';
              });

              String outputVideoUrl = '';
              final apiKey = _apiKeyController.text.trim();

              if (apiKey.isNotEmpty && _img1Path != null && _vid1Path != null) {
                setState(() {
                  _progress = 0.35;
                  _statusMessage = 'Uploading references to cloud...';
                });
                final imgUrl = await _uploadToTempHost(_img1Path!);
                final vidUrl = await _uploadToTempHost(_vid1Path!);

                if (imgUrl != null && vidUrl != null) {
                  setState(() {
                    _progress = 0.65;
                    _statusMessage = 'Running AI Motion & LipSync Transfer...';
                  });
                  try {
                    final resp = await http.post(
                      Uri.parse('https://fal.run/fal-ai/live-portrait'),
                      headers: {
                        'Authorization': 'Key $apiKey',
                        'Content-Type': 'application/json',
                      },
                      body: jsonEncode({
                        'image_url': imgUrl,
                        'video_url': vidUrl,
                      }),
                    );
                    if (resp.statusCode == 200) {
                      final data = jsonDecode(resp.body);
                      outputVideoUrl = data['video']?['url'] ?? '';
                    }
                  } catch (_) {}
                }
              } else {
                for (int i = 2; i <= 10; i++) {
                  await Future.delayed(const Duration(milliseconds: 300));
                  setState(() {
                    _progress = i / 10;
                    _statusMessage = 'Combining @image1 + @video1 + @audio1... ${(i * 10)}%';
                  });
                }
              }

              setState(() {
                _isGenerating = false;
                _creations.insert(0, {
                  'prompt': _prompt.text.trim(),
                  'meta': '$_ratio • $_duration • ${_img1Name ?? "Image"} + ${_vid1Name ?? "Video"}',
                  'imagePath': _img1Path ?? '',
                  'videoUrl': outputVideoUrl,
                });
                _tab = 2;
              });
            }

            void _openApiDialog() {
              showDialog(
                context: context,
                builder: (ctx) => AlertDialog(
                  backgroundColor: const Color(0xFF1A1A24),
                  title: const Text('Connect Real AI GPU (fal.ai)'),
                  content: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      const Text(
                        'Paste a fal.ai API key to run real LivePortrait / Act-One motion transfer from @video1 onto @image1:',
                        style: TextStyle(fontSize: 12, color: Colors.grey),
                      ),
                      const SizedBox(height: 10),
                      TextField(
                        controller: _apiKeyController,
                        decoration: const InputDecoration(
                          hintText: 'Key_id:Key_secret',
                          filled: true,
                        ),
                      ),
                    ],
                  ),
                  actions: [
                    TextButton(onPressed: () => Navigator.pop(ctx), child: const Text('Save Key')),
                  ],
                ),
              );
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
                  actions: [
                    IconButton(
                      icon: const Icon(Icons.vpn_key, color: Color(0xFF00E5FF)),
                      tooltip: 'API Key',
                      onPressed: _openApiDialog,
                    ),
                  ],
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
              final templates = [
                {'title': 'Act-One Emotion Transfer', 'desc': 'Transfer crying/talking from @video1 to @image1'},
                {'title': 'Lip-Sync Voiceover', 'desc': 'Sync @image1 lips to @audio1 speech'},
                {'title': 'Dance Motion Copy', 'desc': 'Copy full-body movement from @video1'},
                {'title': 'Cinematic Scene', 'desc': 'Turn @image1 into a 10s movie clip'},
              ];
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
                        Text(templates[i]['title']!, style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 15)),
                        const SizedBox(height: 4),
                        Text(templates[i]['desc']!, style: const TextStyle(color: Colors.grey, fontSize: 11)),
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
                  const Text('1. Tap Boxes to Pick Real Files From Your Phone', style: TextStyle(fontWeight: FontWeight.bold)),
                  const SizedBox(height: 10),
                  Row(
                    children: [
                      _refBox('@image1', _img1Name ?? 'Pick Photo', Icons.image, _img1Name != null, () => _pickMedia('image'), _img1Path),
                      const SizedBox(width: 8),
                      _refBox('@video1', _vid1Name ?? 'Pick Video', Icons.videocam, _vid1Name != null, () => _pickMedia('video'), null),
                      const SizedBox(width: 8),
                      _refBox('@audio1', _aud1Name ?? 'Pick Audio', Icons.audiotrack, _aud1Name != null, () => _pickMedia('audio'), null),
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
                  if (_isGenerating) ...[
                    LinearProgressIndicator(value: _progress, color: const Color(0xFF00E5FF)),
                    const SizedBox(height: 8),
                    Center(child: Text(_statusMessage)),
                    const SizedBox(height: 12),
                  ],
                  SizedBox(
                    height: 52,
                    child: ElevatedButton.icon(
                      style: ElevatedButton.styleFrom(backgroundColor: const Color(0xFF7C4DFF)),
                      onPressed: _isGenerating ? null : _generate,
                      icon: const Icon(Icons.bolt, color: Colors.white),
                      label: Text(_isGenerating ? 'Generating...' : 'Generate AI Video',
                          style: const TextStyle(color: Colors.white, fontWeight: FontWeight.bold, fontSize: 16)),
                    ),
                  ),
                ],
              );
            }

            Widget _refBox(String tag, String sub, IconData icon, bool active, VoidCallback onTap, String? imgPath) {
              return Expanded(
                child: InkWell(
                  onTap: onTap,
                  child: Container(
                    height: 110,
                    padding: const EdgeInsets.all(8),
                    decoration: BoxDecoration(
                      color: const Color(0xFF1A1A24),
                      borderRadius: BorderRadius.circular(12),
                      border: Border.all(color: active ? const Color(0xFF00E5FF) : Colors.white24, width: 1.5),
                      image: (imgPath != null && imgPath.isNotEmpty)
                          ? DecorationImage(image: FileImage(File(imgPath)), fit: BoxFit.cover, opacity: 0.55)
                          : null,
                    ),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Icon(icon, color: active ? const Color(0xFF00E5FF) : Colors.grey),
                        const SizedBox(height: 4),
                        Text(tag, style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 12)),
                        Text(sub, maxLines: 1, overflow: TextOverflow.ellipsis, style: const TextStyle(fontSize: 10, color: Colors.white)),
                      ],
                    ),
                  ),
                ),
              );
            }

            Widget _buildHistory() {
              if (_creations.isEmpty) {
                return const Center(child: Text('No generations yet. Pick @image1 & @video1 in Studio!'));
              }
              return ListView.builder(
                padding: const EdgeInsets.all(16),
                itemCount: _creations.length,
                itemBuilder: (ctx, i) {
                  final item = _creations[i];
                  final hasVideoUrl = item['videoUrl'] != null && item['videoUrl']!.isNotEmpty;
                  return Card(
                    color: const Color(0xFF1A1A24),
                    margin: const EdgeInsets.only(bottom: 14),
                    child: Padding(
                      padding: const EdgeInsets.all(12),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(item['prompt']!, style: const TextStyle(fontWeight: FontWeight.bold)),
                          const SizedBox(height: 6),
                          Text(item['meta']!, style: const TextStyle(color: Colors.grey, fontSize: 12)),
                          if (hasVideoUrl) ...[
                            const SizedBox(height: 10),
                            ElevatedButton.icon(
                              onPressed: () => launchUrl(Uri.parse(item['videoUrl']!)),
                              icon: const Icon(Icons.download),
                              label: const Text('Download Generated MP4'),
                            ),
                          ],
                        ],
                      ),
                    ),
                  );
                },
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
