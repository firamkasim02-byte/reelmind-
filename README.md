import 'package:flutter/material.dart';

void main() {
  runApp(const ReelMindApp());
}

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
      home: const MainNavigationScreen(),
    );
  }
}

class GenerationJob {
  final String id;
  final String prompt;
  final String aspectRatio;
  final String duration;
  final List<String> referencesUsed;
  String status;

  GenerationJob({
    required this.id,
    required this.prompt,
    required this.aspectRatio,
    required this.duration,
    required this.referencesUsed,
    this.status = 'Completed',
  });
}

class MainNavigationScreen extends StatefulWidget {
  const MainNavigationScreen({super.key});

  @override
  State<MainNavigationScreen> createState() => _MainNavigationScreenState();
}

class _MainNavigationScreenState extends State<MainNavigationScreen> {
  int _selectedIndex = 1;
  final List<GenerationJob> _jobs = [
    GenerationJob(
      id: '1',
      prompt: 'Animate @image1 character dancing to the beat of @audio1 with camera movement from @video1',
      aspectRatio: '9:16',
      duration: '5s',
      referencesUsed: ['@image1', '@video1', '@audio1'],
    ),
  ];

  void _addNewJob(GenerationJob job) {
    setState(() {
      _jobs.insert(0, job);
      _selectedIndex = 2;
    });
  }

  @override
  Widget build(BuildContext context) {
    final screens = [
      HomeScreen(onOpenStudio: () => setState(() => _selectedIndex = 1)),
      CreationStudioScreen(onCreateJob: _addNewJob),
      HistoryScreen(jobs: _jobs),
    ];

    return Scaffold(
      appBar: AppBar(
        backgroundColor: const Color(0xFF14141D),
        elevation: 0,
        title: const Row(
          children: [
            Icon(Icons.auto_awesome, color: Color(0xFF00E5FF)),
            SizedBox(width: 8),
            Text('ReelMind AI', style: TextStyle(fontWeight: FontWeight.bold)),
          ],
        ),
      ),
      body: SafeArea(child: screens[_selectedIndex]),
      bottomNavigationBar: BottomNavigationBar(
        backgroundColor: const Color(0xFF14141D),
        selectedItemColor: const Color(0xFF7C4DFF),
        unselectedItemColor: Colors.grey,
        currentIndex: _selectedIndex,
        onTap: (index) => setState(() => _selectedIndex = index),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.explore), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.movie_filter), label: 'Studio'),
          BottomNavigationBarItem(icon: Icon(Icons.video_library), label: 'Creations'),
        ],
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  final VoidCallback onOpenStudio;
  const HomeScreen({super.key, required this.onOpenStudio});

  @override
  Widget build(BuildContext context) {
    return ListView(
      padding: const EdgeInsets.all(16),
      children: [
        const Text(
          'Create Viral AI Videos',
          style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 8),
        const Text(
          'Combine @image1, @video1, and @audio1 into one AI creation.',
          style: TextStyle(color: Colors.grey),
        ),
        const SizedBox(height: 20),
        Row(
          children: [
            Expanded(
              child: _FeatureCard(
                title: 'Multi-Ref Video',
                subtitle: '@image1 + @video1 + @audio1',
                icon: Icons.movie_creation_outlined,
                color: const Color(0xFF7C4DFF),
                onTap: onOpenStudio,
              ),
            ),
            const SizedBox(width: 12),
            Expanded(
              child: _FeatureCard(
                title: 'Image to Video',
                subtitle: 'Animate any photo',
                icon: Icons.motion_photos_on,
                color: const Color(0xFF00B8D4),
                onTap: onOpenStudio,
              ),
            ),
          ],
        ),
        const SizedBox(height: 24),
        const Text(
          'Trending Templates',
          style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 12),
        GridView.count(
          shrinkWrap: true,
          physics: const NeverScrollableScrollPhysics(),
          crossAxisCount: 2,
          mainAxisSpacing: 12,
          crossAxisSpacing: 12,
          childAspectRatio: 0.8,
          children: [
            _TemplateCard(title: 'Cyberpunk Action', tag: 'Style Transfer', onTap: onOpenStudio),
            _TemplateCard(title: 'Product Commercial', tag: 'Ad Template', onTap: onOpenStudio),
            _TemplateCard(title: 'Dance Motion Sync', tag: 'Motion + Audio', onTap: onOpenStudio),
            _TemplateCard(title: 'Cinematic Story', tag: 'Storytelling', onTap: onOpenStudio),
          ],
        ),
      ],
    );
  }
}

class _FeatureCard extends StatelessWidget {
  final String title;
  final String subtitle;
  final IconData icon;
  final Color color;
  final VoidCallback onTap;

  const _FeatureCard({
    required this.title,
    required this.subtitle,
    required this.icon,
    required this.color,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(16),
      child: Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: const Color(0xFF1A1A24),
          borderRadius: BorderRadius.circular(16),
          border: Border.all(color: color, width: 1.2),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Icon(icon, color: color, size: 32),
            const SizedBox(height: 12),
            Text(title, style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
            const SizedBox(height: 4),
            Text(subtitle, style: const TextStyle(color: Colors.grey, fontSize: 12)),
          ],
        ),
      ),
    );
  }
}

class _TemplateCard extends StatelessWidget {
  final String title;
  final String tag;
  final VoidCallback onTap;

  const _TemplateCard({required this.title, required this.tag, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(16),
      child: Container(
        decoration: BoxDecoration(
          color: const Color(0xFF1A1A24),
          borderRadius: BorderRadius.circular(16),
          border: Border.all(color: Colors.white10),
        ),
        padding: const EdgeInsets.all(12),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          mainAxisAlignment: MainAxisAlignment.end,
          children: [
            const Expanded(
              child: Center(
                child: Icon(Icons.play_circle_fill, size: 44, color: Colors.white38),
              ),
            ),
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
              decoration: BoxDecoration(
                color: const Color(0xFF2A1F4A),
                borderRadius: BorderRadius.circular(8),
              ),
              child: Text(tag, style: const TextStyle(fontSize: 10, color: Color(0xFFB388FF))),
            ),
            const SizedBox(height: 6),
            Text(title, style: const TextStyle(fontWeight: FontWeight.bold)),
          ],
        ),
      ),
    );
  }
}

class CreationStudioScreen extends StatefulWidget {
  final Function(GenerationJob) onCreateJob;
  const CreationStudioScreen({super.key, required this.onCreateJob});

  @override
  State<CreationStudioScreen> createState() => _CreationStudioScreenState();
}

class _CreationStudioScreenState extends State<CreationStudioScreen> {
  final TextEditingController _promptController = TextEditingController();

  String? _image1Name;
  String? _video1Name;
  String? _audio1Name;

  String _selectedAspectRatio = '9:16';
  String _selectedDuration = '5s';

  void _selectReference(String type) {
    setState(() {
      if (type == 'image') _image1Name = 'character_ref.png';
      if (type == 'video') _video1Name = 'camera_motion.mp4';
      if (type == 'audio') _audio1Name = 'voiceover_beat.mp3';
    });
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Selected @$type reference!')),
    );
  }

  void _insertTag(String tag) {
    final text = _promptController.text;
    final newText = '$text $tag ';
    _promptController.text = newText;
    _promptController.selection = TextSelection.collapsed(offset: newText.length);
  }

  void _submitGeneration() {
    if (_promptController.text.trim().isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Please enter a prompt first!')),
      );
      return;
    }

    final refs = <String>[];
    if (_image1Name != null) refs.add('@image1 ($_image1Name)');
    if (_video1Name != null) refs.add('@video1 ($_video1Name)');
    if (_audio1Name != null) refs.add('@audio1 ($_audio1Name)');

    final newJob = GenerationJob(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      prompt: _promptController.text.trim(),
      aspectRatio: _selectedAspectRatio,
      duration: _selectedDuration,
      referencesUsed: refs,
      status: 'Completed',
    );

    widget.onCreateJob(newJob);
  }

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text('1. Upload References', style: TextStyle(fontWeight: FontWeight.w600)),
          const SizedBox(height: 10),
          Row(
            children: [
              Expanded(
                child: _ReferenceSlot(
                  tag: '@image1',
                  label: _image1Name ?? 'Tap to Select',
                  icon: Icons.image_outlined,
                  isUploaded: _image1Name != null,
                  onTap: () => _selectReference('image'),
                ),
              ),
              const SizedBox(width: 10),
              Expanded(
                child: _ReferenceSlot(
                  tag: '@video1',
                  label: _video1Name ?? 'Tap to Select',
                  icon: Icons.videocam_outlined,
                  isUploaded: _video1Name != null,
                  onTap: () => _selectReference('video'),
                ),
              ),
              const SizedBox(width: 10),
              Expanded(
                child: _ReferenceSlot(
                  tag: '@audio1',
                  label: _audio1Name ?? 'Tap to Select',
                  icon: Icons.audiotrack_outlined,
                  isUploaded: _audio1Name != null,
                  onTap: () => _selectReference('audio'),
                ),
              ),
            ],
          ),
          const SizedBox(height: 24),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              const Text('2. Prompt & Combine', style: TextStyle(fontWeight: FontWeight.w600)),
              Row(
                children: [
                  _TagChip(tag: '@image1', onTap: () => _insertTag('@image1')),
                  const SizedBox(width: 6),
                  _TagChip(tag: '@video1', onTap: () => _insertTag('@video1')),
                  const SizedBox(width: 6),
                  _TagChip(tag: '@audio1', onTap: () => _insertTag('@audio1')),
                ],
              ),
            ],
          ),
          const SizedBox(height: 10),
          TextField(
            controller: _promptController,
            maxLines: 4,
            decoration: InputDecoration(
              hintText: 'Use @image1 character, follow @video1 motion, and sync to @audio1...',
              filled: true,
              fillColor: const Color(0xFF1A1A24),
              border: OutlineInputBorder(
                borderRadius: BorderRadius.circular(14),
                borderSide: BorderSide.none,
              ),
            ),
          ),
          const SizedBox(height: 24),
          const Text('3. Output Settings', style: TextStyle(fontWeight: FontWeight.w600)),
          const SizedBox(height: 12),
          Row(
            children: [
              Expanded(
                child: _SettingSelector(
                  title: 'Aspect Ratio',
                  options: const ['9:16', '16:9', '1:1'],
                  selected: _selectedAspectRatio,
                  onSelected: (val) => setState(() => _selectedAspectRatio = val),
                ),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: _SettingSelector(
                  title: 'Duration',
                  options: const ['5s', '10s'],
                  selected: _selectedDuration,
                  onSelected: (val) => setState(() => _selectedDuration = val),
                ),
              ),
            ],
          ),
          const SizedBox(height: 28),
          SizedBox(
            width: double.infinity,
            height: 54,
            child: ElevatedButton.icon(
              style: ElevatedButton.styleFrom(
                backgroundColor: const Color(0xFF7C4DFF),
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(14)),
              ),
              onPressed: _submitGeneration,
              icon: const Icon(Icons.bolt, color: Colors.white),
              label: const Text(
                'Generate ReelMind Video',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class _ReferenceSlot extends StatelessWidget {
  final String tag;
  final String label;
  final IconData icon;
  final bool isUploaded;
  final VoidCallback onTap;

  const _ReferenceSlot({
    required this.tag,
    required this.label,
    required this.icon,
    required this.isUploaded,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(12),
      child: Container(
        height: 105,
        padding: const EdgeInsets.all(10),
        decoration: BoxDecoration(
          color: const Color(0xFF1A1A24),
          borderRadius: BorderRadius.circular(12),
          border: Border.all(
            color: isUploaded ? const Color(0xFF00E5FF) : Colors.white12,
            width: 1.5,
          ),
        ),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(icon, color: isUploaded ? const Color(0xFF00E5FF) : Colors.grey),
            const SizedBox(height: 6),
            Text(
              tag,
              style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 12, color: Color(0xFFB388FF)),
            ),
            const SizedBox(height: 4),
            Text(
              label,
              maxLines: 1,
              overflow: TextOverflow.ellipsis,
              style: const TextStyle(fontSize: 10, color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}

class _TagChip extends StatelessWidget {
  final String tag;
  final VoidCallback onTap;

  const _TagChip({required this.tag, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(8),
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
        decoration: BoxDecoration(
          color: const Color(0xFF2A1F4A),
          borderRadius: BorderRadius.circular(8),
        ),
        child: Text('+ $tag', style: const TextStyle(fontSize: 12, color: Color(0xFFB388FF))),
      ),
    );
  }
}

class _SettingSelector extends StatelessWidget {
  final String title;
  final List<String> options;
  final String selected;
  final Function(String) onSelected;

  const _SettingSelector({
    required this.title,
    required this.options,
    required this.selected,
    required this.onSelected,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: const Color(0xFF1A1A24),
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(title, style: const TextStyle(fontSize: 12, color: Colors.grey)),
          const SizedBox(height: 8),
          Wrap(
            spacing: 6,
            children: options.map((opt) {
              final isSelected = opt == selected;
              return ChoiceChip(
                label: Text(opt, style: const TextStyle(fontSize: 12)),
                selected: isSelected,
                selectedColor: const Color(0xFF7C4DFF),
                onSelected: (_) => onSelected(opt),
              );
            }).toList(),
          ),
        ],
      ),
    );
  }
}

class HistoryScreen extends StatelessWidget {
  final List<GenerationJob> jobs;
  const HistoryScreen({super.key, required this.jobs});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      padding: const EdgeInsets.all(16),
      itemCount: jobs.length,
      itemBuilder: (context, index) {
        final job = jobs[index];
        return Container(
          margin: const EdgeInsets.only(bottom: 14),
          padding: const EdgeInsets.all(16),
          decoration: BoxDecoration(
            color: const Color(0xFF1A1A24),
            borderRadius: BorderRadius.circular(16),
          ),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Container(
                height: 160,
                decoration: BoxDecoration(
                  color: Colors.black38,
                  borderRadius: BorderRadius.circular(12),
                ),
                child: const Center(
                  child: Icon(Icons.play_circle_outline, size: 54, color: Colors.white70),
                ),
              ),
              const SizedBox(height: 12),
              Text(job.prompt, style: const TextStyle(fontWeight: FontWeight.w600)),
              const SizedBox(height: 8),
              Wrap(
                spacing: 8,
                children: [
                  Chip(label: Text('Ratio: ${job.aspectRatio}')),
                  Chip(label: Text('Length: ${job.duration}')),
                  ...job.referencesUsed.map((r) => Chip(label: Text(r))),
                ],
              ),
            ],
          ),
        );
      },
    );
  }
}
