# qwer
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:font_awesome_flutter/font_awesome_flutter.dart';
import 'package:image_picker/image_picker.dart';
import 'dart:convert';
import 'package:shared_preferences/shared_preferences.dart'; // 데이터 저장 패키지
import 'package:intl/intl.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: MainScreen(),
    );
  }
}

class MainScreen extends StatefulWidget {
  @override
  _MainScreenState createState() => _MainScreenState();
}

class _MainScreenState extends State<MainScreen> {
  int _selectedIndex = 0;
  bool _isLoggedIn = false; // 로그인 상태

  final List<Widget> _screens = [
    PlantScreen(), // Home 화면
    CommunityScreen(), // Community 화면 추가
    ListScreen(),
    ProfileScreen(), // Profile 화면 추가
  ];

  @override
  void initState() {
    super.initState();
    // 로그인 다이얼로그 호출
    WidgetsBinding.instance.addPostFrameCallback((_) {
      _showLoginDialog(); // 다이얼로그 호출
    });
  }

  void _onItemTapped(int index) {
    setState(() {
      _selectedIndex = index;
    });
  }

  void _showLoginDialog() {
    TextEditingController usernameController = TextEditingController();
    TextEditingController passwordController = TextEditingController();

    showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: Text("로그인"),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: usernameController,
                decoration: InputDecoration(labelText: "아이디"),
              ),
              TextField(
                controller: passwordController,
                decoration: InputDecoration(labelText: "비밀번호"),
                obscureText: true,
              ),
            ],
          ),
          actions: [
            TextButton(
              onPressed: () {
                // 로그인 버튼 클릭 시
                if (usernameController.text == "무럭무럭" &&
                    passwordController.text == "mrmr") {
                  setState(() {
                    _isLoggedIn = true; // 로그인 성공 시 상태 변경
                  });
                  Navigator.pop(context); // 다이얼로그 닫기
                } else {
                  // 잘못된 로그인 정보 처리
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text("아이디 또는 비밀번호가 잘못되었습니다.")),
                  );
                }
              },
              child: Text("로그인"),
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    // 로그인하지 않은 경우
    if (!_isLoggedIn) {
      return Container(); // 빈 화면 반환
    }

    return Scaffold(
      body: _screens[_selectedIndex],
      bottomNavigationBar: BottomNavigationBar(
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(
            icon: Icon(Icons.home, size: 30), // Home 아이콘
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.calendar_today, size: 30), // Calendar 아이콘
            label: 'Community',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.list, size: 30), // List 아이콘
            label: 'List',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person, size: 30), // Profile 아이콘
            label: 'Profile',
          ),
        ],
        currentIndex: _selectedIndex,
        selectedItemColor: Colors.green, // 선택된 아이콘 색상
        unselectedItemColor: Colors.grey, // 선택되지 않은 아이콘 색상
        selectedFontSize: 14, // 선택된 라벨 폰트 크기
        unselectedFontSize: 12, // 선택되지 않은 라벨 폰트 크기
        showUnselectedLabels: true, // 선택되지 않은 라벨 표시
        onTap: _onItemTapped,
        type: BottomNavigationBarType.fixed, // 고정형 바 설정
      ),
    );
  }
}

class PlantScreen extends StatefulWidget {
  @override
  _PlantScreenState createState() => _PlantScreenState();
}

class _PlantScreenState extends State<PlantScreen> {
  List<String> diaries = []; // 일기를 저장할 리스트
  File? _selectedImage; // 선택된 이미지를 저장할 변수
  DateTime? startDate; // D+ 시작 날짜

  @override
  void initState() {
    super.initState();
    _loadData(); // 앱 시작 시 저장된 데이터를 불러옴
  }

  // 저장된 데이터를 불러오는 함수
  Future<void> _loadData() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();

    // 시작 날짜 불러오기
    String? savedStartDate = prefs.getString('startDate');
    if (savedStartDate != null) {
      setState(() {
        startDate = DateTime.parse(savedStartDate);
      });
    }

    // 일기 리스트 불러오기
    List<String>? savedDiaries = prefs.getStringList('diaries');
    if (savedDiaries != null) {
      setState(() {
        diaries = savedDiaries;
      });
    }
  }

  // 시작 날짜를 저장하는 함수
  Future<void> _saveStartDate(date) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setString('startDate', date.toIso8601String());
  }

  // 일기 리스트를 저장하는 함수
  Future<void> _saveDiaries() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setStringList('diaries', diaries);
  }

  // D+ 날짜 계산
  int _calculateDaysPassed() {
    if (startDate == null) return 0;
    final DateTime today = DateTime.now();
    return today.difference(startDate!).inDays; // 시작 날짜와 현재 날짜의 차이를 계산
  }

  // 이미지를 선택하는 함수
  Future<void> _pickImage() async {
    final ImagePicker _picker = ImagePicker();
    final XFile? image = await _picker.pickImage(source: ImageSource.gallery); // 갤러리에서 이미지 선택

    if (image != null) {
      setState(() {
        _selectedImage = File(image.path); // 선택한 이미지 파일을 저장
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFFFFEFE),
      body: SafeArea(
        child: Column(
          children: [
            const Spacer(flex: 1),
            const Text(
              'WITH PLANT',
              style: TextStyle(
                fontSize: 28,
                fontWeight: FontWeight.bold,
                color: Color(0xFF7B9569),
              ),
            ),
            const SizedBox(height: 10),
            Container(
              padding: const EdgeInsets.symmetric(vertical: 5, horizontal: 20),
              decoration: BoxDecoration(
                color: const Color(0xFFEFF1EC),
                borderRadius: BorderRadius.circular(20),
                border: Border.all(color: const Color(0xFF7B9569)),
              ),
              child: Text(
                'D+48',
                style: const TextStyle(
                  fontSize: 16,
                  color: Color(0xFF7B9569),
                ),
              ),
            ),
            const SizedBox(height: 10),
            Container(
              padding: const EdgeInsets.all(20),
              margin: const EdgeInsets.symmetric(horizontal: 40),
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(20),
                boxShadow: [
                  BoxShadow(
                    color: Colors.grey.withOpacity(0.3),
                    spreadRadius: 2,
                    blurRadius: 5,
                    offset: const Offset(0, 3),
                  ),
                ],
              ),
              child: const Column(
                children: [
                  Text(
                    '목이 말라요...',
                    style: TextStyle(
                      fontSize: 18,
                      color: Colors.black,
                    ),
                  ),
                  SizedBox(height: 5),
                  Text(
                    '저를 클릭해서 퀘스트를 달성해주세요!',
                    style: TextStyle(
                      fontSize: 14,
                      color: Colors.black,
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            Row(
              children: [
                Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    _buildIconWithText(Icons.store, '상점'),
                    const SizedBox(height: 10),
                    _buildIconWithText(Icons.book, '일기', _writeDiary),
                  ],
                ),
                const Spacer(),
                Column(
                  crossAxisAlignment: CrossAxisAlignment.end,
                  children: [
                    Chip(
                      label: const Text('새싹이'),
                      backgroundColor: const Color(0xFFEFF1EC),
                      side: const BorderSide(color: Color(0xFF7B9569)),
                    ),
                    const SizedBox(height: 10),
                    Chip(
                      label: const Text('툰토이'),
                      backgroundColor: const Color(0xFFEFF1EC),
                      side: const BorderSide(color: Color(0xFF7B9569)),
                    ),
                    const SizedBox(height: 10),
                    Chip(
                      label: const Text('별이'),
                      backgroundColor: const Color(0xFFEFF1EC),
                      side: const BorderSide(color: Color(0xFF7B9569)),
                    ),
                  ],
                ),
              ],
            ),



            // Stack으로 이미지 배치
            Expanded(
              child: Stack(
                clipBehavior: Clip.none, // Stack
                children: [
                  // 새싹&흙 이미지
                  Positioned(
                    top: 50, // 바닥에서 약간 위에 위치
                    left: 0,
                    right: 0,
                    child: Image.asset(
                      'assets/새싹&흙.png',
                      width: 300, // 크기 조정
                      height: 300,
                    ),
                  ),
                  // 말풍선&비 이미지
                  Positioned(
                    top: -50, // 새싹&흙 이미지 위에 위치
                    left: 0,
                    right: 0,
                    child: Image.asset(
                      'assets/말풍선&비.png',
                      width: 150, // 크기 조정
                      height: 150,
                    ),
                  ),
                ],
              ),
            ),

            const SizedBox(height: 10),

            Expanded(
              child: ListView.builder(
                itemCount: diaries.length,
                itemBuilder: (context, index) {
                  return ListTile(
                    title: Text(diaries[index]),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }

// 아이콘과 텍스트를 결합하는 함수
  Widget _buildIconWithText(IconData icon, String text, [VoidCallback? onTap]) {
    return GestureDetector(
      onTap: onTap, // 클릭 시 동작
      child: Row(
        children: [
          Icon(icon, size: 30, color: const Color(0xFF7B9569)),
          const SizedBox(width: 8),
          Text(
            text,
            style: const TextStyle(
              fontSize: 18,
              color: Color(0xFF7B9569),
            ),
          ),
        ],
      ),
    );
  }
  // 일기 작성 다이얼로그 띄우기
  void _writeDiary() {
    TextEditingController titleController = TextEditingController();
    TextEditingController diaryController = TextEditingController();
    DateTime selectedDate = DateTime.now();

    showDialog(
      context: context,
      builder: (context) {
        return StatefulBuilder(
          builder: (context, setState) {
            return AlertDialog(
              contentPadding: const EdgeInsets.all(0),
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(15),
              ),
              content: Container(
                width: double.maxFinite,
                padding: const EdgeInsets.all(20),
                decoration: BoxDecoration(
                  color: const Color(0xFFEFF1EC), // 연한 녹색 배경
                  borderRadius: BorderRadius.circular(20),
                ),
                child: Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    const Text(
                      'Add diary', // 다이얼로그 제목
                      style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 20),

                    // 이미지 업로드 섹션
                    GestureDetector(
                      onTap: _pickImage,
                      child: Container(
                        height: 150,
                        width: double.infinity,
                        decoration: BoxDecoration(
                          color: Colors.grey[300],
                          borderRadius: BorderRadius.circular(12),
                        ),
                        child: _selectedImage != null
                            ? Image.file(
                          _selectedImage!,
                          fit: BoxFit.cover,
                        )
                            : const Icon(
                          Icons.add_photo_alternate,
                          size: 50,
                          color: Colors.grey,
                        ),
                      ),
                    ),

                    const SizedBox(height: 20),

                    // 날짜 선택 섹션
                    GestureDetector(
                      onTap: () async {
                        DateTime? pickedDate = await showDatePicker(
                          context: context,
                          initialDate: selectedDate,
                          firstDate: DateTime(2000),
                          lastDate: DateTime(2100),
                        );
                        if (pickedDate != null) {
                          setState(() {
                            selectedDate = pickedDate;
                          });
                        }
                      },
                      child: Container(
                        padding: const EdgeInsets.symmetric(vertical: 15),
                        decoration: BoxDecoration(
                          borderRadius: BorderRadius.circular(12),
                          color: Colors.white,
                          border: Border.all(color: Colors.grey[300]!),
                        ),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            const Icon(Icons.calendar_today, color: Colors.grey),
                            const SizedBox(width: 10),
                            Text(
                              "${selectedDate.year}-${selectedDate.month.toString().padLeft(2, '0')}-${selectedDate.day.toString().padLeft(2, '0')}",
                              style: const TextStyle(fontSize: 16),
                            ),
                          ],
                        ),
                      ),
                    ),

                    const SizedBox(height: 20),

                    // 일기 제목 입력 필드
                    TextField(
                      controller: titleController,
                      decoration: InputDecoration(
                        labelText: '일기 제목을 입력하세요',
                        filled: true,
                        fillColor: Colors.white,
                        border: OutlineInputBorder(
                          borderRadius: BorderRadius.circular(12),
                          borderSide: BorderSide.none,
                        ),
                      ),
                    ),

                    const SizedBox(height: 20),

                    // 일기 내용 입력 필드
                    TextField(
                      controller: diaryController,
                      decoration: InputDecoration(
                        labelText: '내 반려식물이 얼마나 자랐는지 적어주세요',
                        filled: true,
                        fillColor: Colors.white,
                        border: OutlineInputBorder(
                          borderRadius: BorderRadius.circular(12),
                          borderSide: BorderSide.none,
                        ),
                      ),
                      maxLines: 4,
                    ),

                    const SizedBox(height: 20),

                    // 저장 및 취소 버튼
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                      children: [
                        TextButton(
                          onPressed: () {
                            Navigator.pop(context);
                          },
                          child: const Text('취소'),
                        ),
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              String entry = '${DateFormat('yyyy-MM-dd').format(selectedDate)} - ${titleController.text}: ${diaryController.text}';
                              if (_selectedImage != null) {
                                entry += ' (이미지 포함)';
                              }
                              diaries.add(entry);
                              _saveDiaries(); // 일기 저장
                            });
                            Navigator.pop(context);
                          },
                          child: const Text('저장'),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            );
          },
        );
      },
    );
  }
}

class CommunityScreen extends StatefulWidget {
  @override
  _CommunityScreenState createState() => _CommunityScreenState();
}

class _CommunityScreenState extends State<CommunityScreen>
    with AutomaticKeepAliveClientMixin {
  final TextEditingController _textController = TextEditingController();
  File? _selectedImage;
  List<Map<String, dynamic>> _posts = [];
  String _sortType = 'latest';
  final String _currentUserId = 'user123'; // 현재 사용자 ID
  String _searchQuery = '';
  bool _isWritingPost = false; // 글 작성 모드 여부

  @override
  bool get wantKeepAlive => true; // 상태를 유지하기 위해 이 값을 true로 설정

  @override
  void initState() {
    super.initState();
    _loadPosts();
  }

  Future<void> _pickImage() async {
    final pickedFile = await ImagePicker().pickImage(source: ImageSource.gallery);
    if (pickedFile != null) {
      setState(() {
        _selectedImage = File(pickedFile.path);
      });
    }
  }

  Future<void> _savePosts() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    List<String> encodedPosts = _posts.map((post) {
      return jsonEncode({
        'text': post['text'],
        'imagePath': post['image']?.path,
        'likes': post['likes'],
        'likedUsers': post['likedUsers'].toList(),
        'timestamp': post['timestamp'].toIso8601String(),
      });
    }).toList();

    prefs.setStringList('posts', encodedPosts);
  }

  Future<void> _loadPosts() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    List<String>? encodedPosts = prefs.getStringList('posts');

    if (encodedPosts != null) {
      setState(() {
        _posts = encodedPosts.map((encodedPost) {
          Map<String, dynamic> post = jsonDecode(encodedPost);
          return {
            'text': post['text'],
            'image': post['imagePath'] != null ? File(post['imagePath']) : null,
            'likes': post['likes'],
            'likedUsers': Set<String>.from(post['likedUsers']),
            'timestamp': DateTime.parse(post['timestamp']),
          };
        }).toList();
      });
    }
  }

  void _post() {
    if (_textController.text.isNotEmpty || _selectedImage != null) {
      setState(() {
        _posts.add({
          'text': _textController.text,
          'image': _selectedImage,
          'likes': 0,
          'likedUsers': <String>{}, // 좋아요를 누른 사용자 목록 초기화
          'timestamp': DateTime.now(),
        });
        _textController.clear();
        _selectedImage = null;
        _isWritingPost = false; // 게시 후 글 작성 모드 종료
      });
      _savePosts();
    }
  }

  void _likePost(int index) {
    setState(() {
      final post = _posts[index];
      final likedUsers = post['likedUsers'] as Set<String>;

      // 좋아요를 추가하거나 취소
      if (!likedUsers.contains(_currentUserId)) {
        post['likes']++;
        likedUsers.add(_currentUserId);
      } else {
        post['likes']--;
        likedUsers.remove(_currentUserId);
      }
      _savePosts(); // 변경된 내용을 저장
    });
  }

  List<Map<String, dynamic>> _getSortedPosts() {
    List<Map<String, dynamic>> sortedPosts = List.from(_posts);
    if (_sortType == 'popular') {
      sortedPosts.sort((a, b) => b['likes'].compareTo(a['likes']));
    } else {
      sortedPosts.sort((a, b) => b['timestamp'].compareTo(a['timestamp']));
    }
    return sortedPosts;
  }

  // 글 검색 기능
  List<Map<String, dynamic>> _getFilteredPosts() {
    if (_searchQuery.isEmpty) {
      return _getSortedPosts();
    } else {
      return _getSortedPosts().where((post) {
        return post['text'].toString().toLowerCase().contains(_searchQuery.toLowerCase());
      }).toList();
    }
  }

  @override
  Widget build(BuildContext context) {
    super.build(context); // AutomaticKeepAliveClientMixin이 적용되도록 호출
    return Scaffold(
      backgroundColor: const Color(0xFFD3E8D3),
      appBar: AppBar(
        backgroundColor: const Color(0xFFD3E8D3),
        elevation: 0,
        title: const Text(
          '커뮤니티',
          style: TextStyle(color: Colors.black),
        ),
        actions: [
          IconButton(
            icon: const Icon(Icons.notifications, color: Colors.black),
            onPressed: () {},
          ),
          IconButton(
            icon: const Icon(Icons.account_circle, color: Colors.black),
            onPressed: () {},
          ),
        ],
      ),
      // 플로팅 액션 버튼 추가
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          setState(() {
            _isWritingPost = true; // 글 작성 모드로 전환
          });
        },
        child: const Icon(Icons.edit),
      ),
      body: _isWritingPost
          ? Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: TextField(
              controller: _textController,
              decoration: InputDecoration(
                filled: true,
                fillColor: Colors.white,
                prefixIcon: const Icon(Icons.edit, color: Colors.grey),
                hintText: '글을 입력하세요...',
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(30),
                  borderSide: BorderSide.none,
                ),
              ),
              maxLines: 3,
            ),
          ),
          if (_selectedImage != null)
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: Image.file(
                _selectedImage!,
                height: 150,
                fit: BoxFit.cover,
              ),
            ),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16.0),
            child: Row(
              children: [
                IconButton(
                  icon: const Icon(Icons.photo_library),
                  onPressed: _pickImage,
                ),
                const Spacer(),
                ElevatedButton(
                  onPressed: _post,
                  child: const Text('게시하기'),
                ),
              ],
            ),
          ),
        ],
      )
          : Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: TextField(
              controller: _textController,
              decoration: InputDecoration(
                filled: true,
                fillColor: Colors.white,
                prefixIcon: const Icon(Icons.search, color: Colors.grey), // 검색 아이콘 추가
                hintText: '검색하기...', // 글 입력 대신 검색하기로 변경
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(30),
                  borderSide: BorderSide.none,
                ),
              ),
              maxLines: 1,
              onChanged: (value) {
                setState(() {
                  _searchQuery = value;
                });
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16.0),
            child: Row(
              children: [
                ChoiceChip(
                  label: const Text('🔥 인기순'),
                  selected: _sortType == 'popular',
                  onSelected: (bool selected) {
                    setState(() {
                      _sortType = 'popular';
                    });
                  },
                ),
                const SizedBox(width: 8),
                ChoiceChip(
                  label: const Text('📅 최신순'),
                  selected: _sortType == 'latest',
                  onSelected: (bool selected) {
                    setState(() {
                      _sortType = 'latest';
                    });
                  },
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              padding: const EdgeInsets.all(16.0),
              itemCount: _getFilteredPosts().length, // 필터링된 게시글 개수로 설정
              itemBuilder: (context, index) {
                final post = _getFilteredPosts()[index]; // 필터링된 게시글 목록
                return Padding(
                  padding: const EdgeInsets.only(bottom: 16.0),
                  child: Container(
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(20),
                    ),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        if (post['text'] != null)
                          Padding(
                            padding: const EdgeInsets.all(16.0),
                            child: Text(post['text']),
                          ),
                        if (post['image'] != null)
                          Image.file(
                            post['image'],
                            height: 150,
                            width: double.infinity,
                            fit: BoxFit.cover,
                          ),
                        Padding(
                          padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0),
                          child: Row(
                            children: [
                              IconButton(
                                icon: Icon(
                                  post['likedUsers'].contains(_currentUserId)
                                      ? Icons.favorite
                                      : Icons.favorite_border,
                                  color: Colors.red,
                                ),
                                onPressed: () => _likePost(_posts.indexOf(post)),
                              ),
                              Text('${post['likes']}'),
                              const Spacer(),
                              Text(
                                '${post['timestamp'].year}-${post['timestamp'].month}-${post['timestamp'].day}',
                                style: const TextStyle(color: Colors.grey),
                              ),
                            ],
                          ),
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class ListScreen extends StatefulWidget {
  @override
  _ListScreenState createState() => _ListScreenState();
}

bool _checked0Extra = false; // 추가 상태 변수 추가


class _ListScreenState extends State<ListScreen> {
  // 체크박스 상태를 관리하는 리스트
  List<bool> _checked = [false, false, false];
  List<bool> _checked0Extra = [false,false,false];
  List<bool> _checked1Extra = [false,false,false];
  List<bool> _checked2Extra = [false,false,false];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFBBDDB1), // 배경색 설정
      appBar: PreferredSize(
        preferredSize: Size.fromHeight(65.0), // AppBar의 높이를 원하는 크기로 설정
        child: AppBar(
          backgroundColor: const Color(0xFFBBDDB1),
          elevation: 0,
          flexibleSpace: const Padding(
            padding: EdgeInsets.only(left: 16.0, top: 40.0), // 텍스트 위치 조정
            child: Text(
              'NOW님의 온실',
              style: TextStyle(color: Colors.white, fontSize: 20, fontWeight: FontWeight.w900),
            ),
          ),
        ),
      ),
      body: SingleChildScrollView(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 상단 이미지 컨테이너 2개
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: SingleChildScrollView(
                scrollDirection: Axis.horizontal, // 가로 방향으로 스크롤 가능하게 설정
                child: Row(
                  children: [
                    Container(
                      width: 100,
                      height: 100,
                      decoration: BoxDecoration(
                        color: Colors.white,
                        borderRadius: BorderRadius.circular(10),
                        border: Border.all(
                          color: Colors.green,
                          width: 2.0,
                        ),
                        image: DecorationImage(
                          image: NetworkImage(
                            'data:image/jpeg;base64,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', // 여기에 이미지 URL을 입력하세요
                          ),
                          fit: BoxFit.cover, // 이미지를 컨테이너 크기에 맞게 조정
                        ),
                      ),
                    ),
                    SizedBox(width: 16.0), // 두 컨테이너 사이의 간격
                    Container(
                      width: 100,
                      height: 100,
                      decoration: BoxDecoration(
                        color: Colors.white,
                        borderRadius: BorderRadius.circular(10),
                        border: Border.all(
                          color: Colors.green,
                          width: 2.0,
                        ),
                        image: DecorationImage(
                          image: NetworkImage(
                            'data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wCEAAkGBxITEhUTEhIWFRUXGBUVGBcVGBgXGBgXGBUXFxcYFRcYHSggGBolGxUXITEhJSkrLy4uFx8zODMtNygtLisBCgoKDg0OGxAQGy0lHyUtLS0tLS0rLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLf/AABEIAP8AxgMBIgACEQEDEQH/xAAcAAABBQEBAQAAAAAAAAAAAAAGAQIDBAUHAAj/xABJEAACAAMEBgUIBwYEBgMAAAABAgADEQQSITEFBkFRYXETIoGRoQcyQlKSscHRFCMzYnKCslOiwtLh8BVDc4MWJGOTo+J0w/H/xAAaAQACAwEBAAAAAAAAAAAAAAADBAECBQAG/8QAMhEAAgIBAwIFAgUCBwAAAAAAAAECEQMSITEEQRMiMlFhcYEFQpGhwbHxFCQzYtHh8P/aAAwDAQACEQMRAD8AJTHohvzPWHsrHukmfd9n+sB2DaiUQ4RXEyZXzU7j84lWY/qL+9HUcpEsehgmN6g7z8oW+fU/e/pEUW1joSE6T7h7x8o90n3G8I6jtaPVhI90g9V+5fnCCYu5u4fOJpldSPQhjxmL972TCGYu8+y3yiaZGpCGGkwpmJ6w7m+UNMxPXXx+UWoq2jxMITC309de8R6q+svtD5x1HakMJhIcVG8d4+ce6Pl3iJpnWhkJEhlHdCdEdx7og6yNjFWY9O6LcxCPRPcYqOhOyJRDGHGkPWIJkqPS1MEjwDfJX1ln0lp+L4GPRBrCt64v4m9w+MegiKHYDq5L/aP3S/5IQ6tp67ezL/ljaELAQphf8Nr6/eq/CPf8Oj1x7H/tG5CxxBiDV5fWHsn+aFOgF3juPzjaj0cSYn+ADePGGtoAcPH5RuwP64ac+jy7qn6xwafdG1ue7+kUnKMIuT7EMFNadKSrO3RJR5nplTUJwrTzvdzgg1fRbTISYvRtgKhqsyHarE1IP9iuccrtLZmK8jSbyWvS5jI29CVPIkZjhGdi/FJOT28oBpN2zss7QjV6sqWRt67q3ZgQfCA7TescqzzGltYnDr674Hcaq2R3isY9k8oVrQ4zb43MqnxpWKGn9OPbpiu4UELcAQUyJIBqTmTDM+sio3Hn6E6U+G/1Lk/Xhj9nZZC/iW+fEQ/R+sNqnvLl3JNHdFurKUHrMAaHYePCMZ5Js01pc0KxwOArcmXaqCSMRjiBhlBZ5ObMHmtaJhCyrOGa8cFBYE8gALxO7DfCq6nNkmop/wBu5PhVy3YRNq1M9U/u/OIn1bf1D3Rha2+UITQZVnYohwJFelmcAB9mvDzjgMMQaegf8YUAWcukvMB2lso5K15gOAHZD8c6nPTGLa90tissij3CNtW3/Zn2T8oifV1/2R9lv5Y1bBadIH7Vwv8AtoO7rkk8wBGnO0yZYq5B50B99Id8Fg/8REETq6/7M+w/8sI2r7j0D7LfywYaJ1okTnEupSYfNVvToKm4wwPLPDdG4IE9tg8XqVpnMP8Ah+aclPcw+ERzNATlFSG72jqkejrLOJxuZIoaF+HnnPviLoWDgVbb6R3c4JtKWe9MmtvnSh+tj+kRmNI+sPJveIs9gSdg9akJnAYmiNnjmy/KPRLpB7s4ncqjvLfKPRxdHbEtC0FWEO6dfWEUwvVGEMYCkdoB+M+DSELCCFgYyej0ehBHHCxxnWnSpn2iY4NRUqu64potPf2x1+3vdlTGrSiMa7qKYANBaLsUiWtotjoWYBklEhqDMEoMWPgIW6nG8iUe3crJWc9da8fdFS1Ghp/fCCDXHTST7Qzy1otFVb25RTIZY1Oe2BibO3mMt4oQk0nf0Fp3wiVLFNZXdUUrLAZ6vLUgHKiswLk0pRQTlhiIryZxU1FVIhv0hc9sS2zSbTTWZMLECgLUwG4Rattkcml33JtIW3peuTRyAGJJIcjAEUHVNAM8MI0P+LSlhNhEpes5d5lW61SGAYKQSQQBnSgApsgdJzbYPE7IrgVxOe8ZQzh8u5M8j57mtI01OQURlTZeRFVqfiArCtpm0tS9aZxpl9Y48ARGYoO+L9gsZmB6FSyAN0fpuuN5pexitASudCSK0MTLLkeyb/UClbCPVDWl7NNZpwe0S3ABDzGYrSvWQObpONKGmQxG3pejbbo+1n6mc6u3+X0s2WwpnSSzXT2KRt4xxY2YjKJJTUOOFMjx3g7IFj6+cOd1+4ZUtqOta0akNMHS2eYROTrCt0FipqtCoAVhTDDPPaYJdW9JG0WaVOIozrVh94Eq3ZUGOVaP10t0oUE7pBsE5Q9PzCjHtYx0bUfS8mfZwspOjMoBGlk1u4YEN6SnHHeDuhzD1WPNLyvf2DQrsEUITCwyaeqeR90NF2Bs5PGd+mW/80ZJGLngPFo2K4Kd7T27gg+MZQXBuajwrBpi0AWt0u/OmDdd8B/7R6L2jJV61TfzeHRD5x6BuVB1Fs6jLeqiGucIjR8IidsRzEHoQb3NqUeqOQ90OiOzeYvIe6JIXNBcHo8I9Hogkxtc7RcsU87WS5yvkJXsvV7I4TaJ+J2Yx2HynTKWMbjNUH2XPvAji7YmM7rZU0gOR7kTEmI2AFeGJ5bzwjfsOgyydI7KoIBRPTeuTUHmJ9457Kxqm0yrIweVLBoKXpx6oJGNyXLoS3Esx5ZwPF08pK3sgbAbpV2Yx4vwi1pGb0sxnuhQSTQKq58FAAyyGUVjhU7sucDajdLcqMmDJd3vhl0iFQRYlCsWc9JR7sgQVyzidFBwYR0jV3UqzWrRnSIo+lfW0e8fOVzdRhWl0rdGW2sA1rsJWm+gPEbCrDYQRQiIzJxSb4Zfw2lZNoeWGboi11j9mWNELH/LYnzSxyY4VwOYKzT7OVZkdSrKbrKwoyncQeYPEEHIxnotRQwbqn06x3gAbXZVBNDVp1nBYUbaXW6SM9nrwDw1lTS9S/f/ALLxVoF5Qg98lDUm2gUwKSyexnp+o90AV/IjbHRvJRZCFnzTkSksc1BZv1r3RXoU3nX3/oXhydAiK0miNyPuiWK2k2pKc8I9EuQsuGCcrzEP/TnHtaYAP0mKQHVP4vcIvqtJY4Spf7zuYouaJ2ufGCS5AR4MXVnGfNb8XjMI/gj0P1PX7Rt93xLt/EI9CuT1DuN1EO2D+i1B2QjNMGbe6Ltns5ORpHrZLCS2eY9EUFmOJwHAZ8obckIqEuS1YT9WvKJXcDMgczSOLaw+UC0uTLkEyJQqBdwmEVzZx5vJct5gPtNucklnJY7SST3nGA6lZZ5kkfRlo0xITzpq8qj50jIna6WVTjMT/uKx7pd6naY4CJ5O48aRYkaQelAAezdhHWV8e+x1vXHWWz2mxvLlNVqy2xByDqTT+xn2QAaNsImTpa0vXmAujbVhUcqViitoZ3GPVWlcKVrh8+6C/UaQFtRnv1ZciXeZtlWqAOZqKCEsjjPIkSnrY3XuyzrLNBYgibfcMowBBAK47QGWm/vgNnTGY1c1Pu5Rva36ca1Tr7ZCoRdirX37ztgcYwp1Gd5JaY8FJpXsMmGK5/vmYlm1290RqvjA47IoQqrAXrrXbwW9Q3QxFQpbIEjZFpBQg7DHX9SLBImaHaVdBV/pAmby15gG/EFCUOyixytyGAoAOqow2kKAT2nGDdTBQUX7ltFLkPfJbpXo5zSSepNFQNgmLke0YflWLPlL1dCn6VKHVYgTVGQc5TOTGgPEg7TAZoKaVdCDRg1K7myB7DjHb5EyXabOCygpOliqncy4qe+nZE9KvGxSwy7cBlvA+fpqEY7o39TtJiz2qVOJolejmfgfAmu5WuMeCxU1g0a1mtEyzsa3cVY+khFUJ40wPEGM2QTlCXmxT+UwcdmEutuhjJt5kSVvCbdmSlG6YSCo4B1fgFpujrGr2jBZrPLk1BKg3iMixJZiOFSeyB3yayDMs4nThedC8mS7YkSeqSqncHvL+WmyDMLGx0+GCbyr837Bku6EpjFHT7UkOeEaMY2tjUszQ3D1Ime0WY7nqHgkhf8Axhv4oz7Y1JNfuMfExoW4UWZ/qAezKVfhGRp3q2VjulDvI/rF2DRQ1U6sktvZR3S0hYj0e12yqd8xvAsv8MJCs5pSY7CFxR1GwbeyMLXy2XZNyufWI3geaO04/ljcsjUqTlSsAWnLZ073gL14llBwF0YBjuUU7amD5ORVPynL9MijkdsZisGJzOygxOOR7409NzQ8x2WjKDdvbDTMgbq5Z7IoyL2CLgWNKDDDbluFYDYnKrElSyAwJA3VxI34D40jQ0TYBevGrBb2FMGJ6qrQE1N5hEli0SSyXgQDRuwdcD2bg/3BBFZWly6zSKIgLrl1piq/RDHZVS35RvjpPsEjH3MubapKWiXZ5g6GWpCT3ABIepq5FaXQCoO4AnE4El1ztHRM1kl0WXKIY0zmTHUFWb8MsgD/APKcttE1mYk1ZjWtKksT4kkwc6d0dPkOsm0G86IgveslwBDxIusn+3Cud6MbaJhJtOjDmEkmPBKRZEqGskZGvsQUytTjDUTDlF3oYbKlRfxERR0PyXWmkm1Sdw6UfmQo1P8Atr3wKSNC10VZ7YoxV5subxUz2WWx4g9Wu5h6samoFpuWgjZMkzU7Qt8foPfG3qRZhM0A8s+klrHbfmUI4g0jTwJZsKi/Zr+gRbxOe2ObdcYAgFTTYcqg90dk1FtN6Q6fs501OwtfH6yOyOMJmDwjqnk7n1mWtfvS5g3UbpMuwCFehlWdfKLQ9LMHysygtqkuBi0oqT+FjT9cC2gtGzLROWVL85znTzR6TNwAx8MyIMfKpLaZarJKRbzlWAUZku4A5eYTXYATBVqdqytjlmpDTnpfYZAZ3Erjd47TjuAZydM8vUP22spW5v2GxpJlpKli6iAKo4DfvO2sWDEYeHA1jRqkMKSZ5npGHrUv1KrnVwN+ZjdMY+sOLSF3zF98TDkjJ6TF0q/UbjMnnucr8IydbsLOw/0177ojS0i1ZafeLH25lfjGVrs31dN81R7Jr8IsUKLClmkjeWbvYn+KPQ+0ikqzrulL+lflHozcr8zNXCvIgx1leYLOVl+mQjnaEIYtTbU0C4ZXq7I59rbpC5L6FWu1AV2FK0AxUfHdUDbgTa46WEs0rktTwqT4nCOW2i/PmVONTgBjhu4Zw3mn5tjGlKlSIZQvmijqriBvOX9/0gl1O0Art0s0VDES1A2IMZrnmAEH+oeYvaJ0KoAlXgCQGmsBW4lMVDbDkN5rBtLVZUvIIoWgBwuotSWO7DZvPKoor3OhDuwZ1okUnDJS6gFRSiKGJbAYZJKX/agV1omBbPLUYNMdphXcqAon7pApwJgjt0zppjTG6obAbAqceSCp4kxsaI1IWfN+kWxeqABKkEUF0EkNPG0kkm5x61fNHU5vYmSb4MDyV6mlmW3WhKICDIVvTb9qR6o9HeccgpJb5RNGCZZzaAtWs9S1MSZLUMygGZW6HH4CPSMGE4DDmBwhZUigIONcCDlSDeGtGlhYwS2RwaZZirEGlQcxipGwg7QQQQdxEILPtMbmkNE9B0tn9KysgXPr2OaSLO2ObS2Bkk7lUmMue4y25R5nqsUsOTR27fQFKOllN1hqpE6oYfcrgO3+sB1diqLWruE+UeJ8VYfGDLybzAmhqnJRaieQmTCYEdHymvi6CSKnqAtTA4kqDdHE4Qmi9JWv6CthlIrdKJihApM6481rzNQ/VocQCwFcaVAJGz+HTcIPUn3/AILIwLDLwUcFFT8Y6f5P0JnT5gBuFJQqcrwFQK5VArUbKisVNW/J8Rde1Mpxr0S4g/icHwHfsjoEqWqqFUBVGACgAAcAMoJ0fRzU1knt8HJ1GimmjE+kNaWFZlwSlr6KAkmnFiTU7gOMX4Y7Qkvbz2xrUVskhTMAGMNrEVoxBG/+g+MdVkqTRPeGBzjK02310jg172QTF+U3W4LnGPrHNpOU+rKnN3Sz845KmWcrj9zLmLUWdfuyR7ifdGHrw9RLG+Y57kc/GCK0Ck6Uu4p+6hMDetHWnWdPxHvZE/iiCUP0wKOq+qgHiY9CaZb65uS+4H4x6MyXJsQ9KNXW3QBtIDy2VXUUIc3VdRUgV9FgSaE4GpB2ECWgLEZc67MF18rrim3bwzNRgaYR1F1wiObZbxAdVI40POm0RoZun1PVF1/Jgt7mbIskiUoIzBLOxAIwNOsU2g4Ko4YYEjD0ra2nOJcsG6xwUCruBiCaV6u2g5ngZy9GSi7KVN2gal5hiTXfvJMaNlskuXXo0Va50GJ5nM9sDeJ8BkrRg6vas3CJk8AuMVTMJtqd7eA8YIWcBuwfGJIhmgVqxoKb6UgsIJbIs6S2IbU1cNhNKRblHZuw/usVZjIQKGtGFYt3hFpcHR55A7yiWK50VtAqJV6RaAM2sk4gTO1GCON1DF6yalWIKCUM053mdusTtopC04ARu23o2VpcwVVwVYHIhhQjuMD2oVqZZcyxTWrMsb9FU5tJIvWd+2XQflgEsUZNOSsJcWWV0FZVrSzSdoxQH3wPeUVUSyyHVFUSbXZJhugAUEymQ5iDIocSCCKnI17IEvKlZy2i7VvVZbj8s6W3uBg/hwUdkL20EmlSOhnUIoJU3I7AhzxjC8nIJ0bLOZJnEnaaTnpU7cBE9qnA2G0TK4GzTGH5pJPxiHycqRoyVznU5dM8Q/8AVX0O/KFMpwRhswhb0Q2UUBrtNYkLDfBGtwdiMYcz0JiJy3oisOIauRzPdSJo5EimGnjuPvhSDwHbEZlA0q1OB3xBZI9tCjt4mMTWHGdMH/QYe0yp8Y27OOtGFpVq2mYP/jr7U9T7lMT3J7EVoP8AzPIOe5KfGBvTGNukLuUH/wAit/DBDWtoc7kmeLKIHpovaTH3UUd6TT8oHLhhI8oj0mazn507gB8I9EVrasxz95v1GFjLfLNhcI6VLAqOcW6UjPEzEc4na2pxMbEoswW0Olfa/k9xi5WM2z2gNNFMOqRGjWKSQTG9h1YrW4dU8viIsVinbRWv4fjHQ9RM3sMlUvdq7QfWEWi8Z8yessF5n1airEndU7s+Ucz1p8oc5yVs56GWMARTpG4lj5o4DvicslHdlNVHVEtFOq45bv6GBHWqWLLbrLbKfVTj9DngVpRutJc7qMMTuw2wHama3z1n/wDMTneSVe+HJcghCylCcbxKhaA0N7lBPMt9m0tZbTZ5N9ZlyoSZQGoIZHWjEUvqoONRXjA45YZFceSFI6DJkhcqKN0DuvQvWK1oBUmzziCN6ozd9VinqZptrVYpUxi3SAdG9TSjy+qxI40Dfmjcnzb0t1dQaq4qMiLpqDBK8up+xZyXCBOVNP8AgjMRQmyBSOKyyh8RF/UdwNHyF3q5PbNc4d8Dptw/wWYtesrmSe2erj91xBNqjLIsNlp+zU99T8YDimsk0/8AbZDTSf1CFZoA8z4wn0o7AB2Q2Ud5iUSwccYYr3IVsb9IY7YivmhrjXfE4Qbu+EuZcaxKo6Ub7kMkYw6YOsTxiVrohAtchE33OpcC2ZcRA/asbS/GdIX2Umv8BBPIlEGBhcbQf9dj2JZyPfMil2ya2RXseMyafur4zCfhGFYOtpGcfVAHcqD+MwQWBftTxljwY/GB7Qp/5q1vuMzwuj+CBy4YWHqRRLVJO8k+MehsuPRlmwdJIiIsu4+A+MTEQ5LG3qf34Ru2YGmxtiI6VaCnne6NiM2TZWV1Y4bKV4UjRECychIbIdFS2sBUkgC6ak5DEYmLVIA9bNMs7XFBEsGgNPPO/iNw7eQJ5VjVsmfBma5aU+km4CRLG7C9xPP+9tQe0aEZ8Q4C7zt/DStYIZpGbmpONM+1j8Buis7lmO4DD++yMyfUyb5ASVmBYtFTCwVSihuqCWNBvJNKjDcIJNAWB9H6QDM6zAqN0hl1NJbJeJBIxu0Vj+GIpEoZnZD1twEwuWuswpgxHVNBQ8DgKbqdtIdS21GPvYTFj1OjV0O30TS9os4IEq00ny9xLi9UdpmDkkHr4K1c6EdvKOL6UnGW0maKk2dwmP7PzkXH7t9YL7JMEuctB1SysG3pMxr3NSsMS6uSjOFepNr47P8AgPHDc2r4B1bWegnSNjWgP7Am1rx+z9mOqatWdxZLMBskyv0AxxmZMpOnjdMmd+MFknSjS1UCa6YKoozAZDDOK9H1Glb+yJjheS6OlLIfhEnRv/ZgHsmtFoX0w43OB7xQxv6O1rltQTVKHeOsvfmI049QpA5YJRNgo++HLKbbE6OGFQQQdohYJrB0Ry5AGeMS1pCAwwnGK88kN0tiVGgSkfak8bY3cZC/OCyXt5QH2M1LHdKnH27QR/8AXHItykS2AdVjvceCD5wLaBPUtb7zM8Xmj4iCix4Sq/emnuoPhAtoQ/8AJzm9Yj95lb+KB5PSw2FedFRIWEWFjNNU6aVjVl5DkPdGbSNGT5o5CNjIY8Bk3ZzEPrDZ2XaPfDop2O7mZp/SBly7qULuDSuSqB1nPAVA5kRl2qQhsoE6tADdahvliCS4qKipIAGZ3ZAEEyyS2YsyhjSnWFQBuAOGZMQ2qxyzQdGmOGQG0HZxAPZC7wzlJu+1JHNnN7bowqyqcCRXhTcP73RALDTLbx+Jwjo2kNBS5qOo6swrdWZUkrTEUqcBhs2RyPTGhLXLmN9Ya4tdY3gQKk9C5G4HEYCmNDQRkz/Dpue86XxyXhis3foAlqGajg4FVIqA2AfcRWnhmDFZLfOtFU6JaJjWoBXebxFEalchU7MMiX6HLlojOQl5QoBHmo1AoNc61Au8Ym0nYrPJl3AxlKCRW/1i167eNT1jWndTdGnhx4unjph9/n6stHHZzvWWRRSTdo6EBgb9Wlm+MbxpW6FphSuWUTWPSQNllFiaKDLwAbEG8uFDQ3TmNiGGnR1pmSyFlMEJDU9K/wBa6QMxWpypTCpyBqaB0fOlo8udKIVgCq3lqWVqrmcN241pvEL5EnNNL/z2f/JzhKMrKBe9PnH1phPYafODFpyAgsUOOHWAF2mw57Gww2QIrY3FoKspxKkkZbAwqaCoocIJ7Oj4glyTQ9XEqMaksThgCN+AO6K4IOLf2DYY1diS7PUdRLpNSL4yrVrpIYZiuRIrkd0ikqaAmu0NWmdM8iSRWldsSCVVRhQdbFQpXCl6lCDW6akkk4GtIWVZVINzA45MGGFclBKA4EEAZHEVMMUFZvao6Y69wHAmhU1wJAIK7wajLfBuwvDCOaaBsolzBaSLktKkZgzWpgEUUF0NQl6Y3eME9g1kJQKsibMfEkqpu1LE5gHfDWJ/l7iWaNLV2CNUIh5UwPNpO3N5shJQ3zGFe6tfCIXS1sOva1Uf9Ja+IAg1MW2CgGgJOGEBejXqHOf1EjvabPb4iK9qk2atJtqmTTu6Rf0irQ6VNlqG6CTOYMEU0VqUSt3GZdG098Q1RZW6LLTLtmLbkmt4sfhA5owXdGj7zSx3AfyRpW9LXMkmVKkS1BQyy8ybWla1NyUrY474p22V0VlSSXRmD1N07KORhmPOAxhfLJaaGsEJa7ozFaPQxTHoRo0TrNIvSfNEU6Rak5Rrz4MiL3FmjCPQr5R6kDOfJ6GTaYVp2w2ba5a+c6jmQIy7brFZVGM0Gm4V99ItFNshtJGtZznl2CkYVm0U0wzASAgdxQi9UgkYA5Ybc4t6A01KtBcSiTcu1y21pSn4Y1klgZbye0mp8TAckfMw2N+UyDq3IrUiv4hexwNanHMVzrjnTCMq2auzwwmShIMyrEuQa0K0IUEGm7zjma1rBcYhcwCSXsGjJgDpDQlvb/LL5jF0ApkvUXClKbewRnDV61lWDyGIOzIqMKUKk5Y1AGPGOg2qdQbYxLbb2Fesw7YG3H2DRTYNJqzac+iZWoFqCKUBHWq487PYKClMThdXQrjCZcSles8yWa3s/NatcxWnziC0aQmN6bU5mKpcwPxUuEH8FvlmjOsknHpJwbYoRS1N+LXR4ke6I/pMpB1Jd5vXnEO2da3aBcMMwchziiTDYo88yywQXyPtdpZzedix3k1ggsUy09EgHRogVQCTMckUzN26FPA1gXeHpYbRNylTXGwlWu04M2HjBemySi21uC6rHGSSeyN+daFH2lr7E6MeKBnihPt1k2q838d5/F2HuhsjVS1NmqJ+Jx/BWNCRqSf8yeOSJ8SfhDerNLhCf+XjyzM/x+7hLkqvP/0CnxivO03Pb0gOSj3tU+MEc3U+QFNHm3t5KmnYFFRAnb7E8prrZbGGR+R4QHLDLFXLgNhyYZuojZ1pd/Odm5sT74qzHh96K01sYAt2MUPBhIh6cA5iEi5B1e0602ZPSryAHiYrjWpn+xs7vxAZh3gUirK0lZ0+wsxrvSTdPtzbte+J5ukrWcRJCr60yZT91VofbjWpGLuOa1aRfKWkob3ZR8zEaaOtD/a2ocpYZ/E090QXLS+LTlUb5Uv3s/SDxEU56yB9tPaZvV5uH/bQuP3RHWkdTfYt2jR1iln66cxO6ZNWWDyAoYRLTYk+ys4Y7D0TNXk82invjOl6SssoUlS/YS6O9mA/diJ9ZGHmSwvNj7kCCKvLFFvCkwosGl5odb0gpKOBZ2VLo3hQCD7WUE6muIjkqaWtEw0lk13SUFe9QW8Y6Pq00w2aV0gYOBdIet7AkAmuOIAPbAJyUnaDwi4qmaZiKYIbMtarg/U4nzTybLvpDhNDDqkHka+6AyCIzrYMIGtK5GCW2mkC+lpuFIWkN4+TEMJCwhMLsbEaGkwjTBv7sfdDCTyillkjT0Bo4zpymnUQhmOzDEKOJNOyvCD+sZOq8m7Zpf3rz+0xI/dpGpG30uJQxr53PO9bmeTK12Ww+9CEw2EJhmhMjn5QOaVQEEEAjjBDOMYGkwKGLSXkGMXIIWyx3TgTd3DMdpjOmy13V5knwyjdtTRkWyWDlgYyJwUWa8JtrcrCeBgKDlhHop4g45x6LKJGs6JN1gI8xLo4tTwQKIotpS0OepX/AG0x9oAnxjVlSJa5Itd9AT3mLSmsW1yfcHUFwgZfRtqm4uGPGY/wJJ8IsSNWXPnTFX8ILe+kEIiSWsFjFPkHKTXBnWbVaQPOZ37Qo/dFfGNWy6Is6ebISu9lvnvapixLESLDWOMfYXnKXuWJZoKDAbhgO6LNlmgkrtFDFMmgJ3CB2ZpboLZKLYKx6NidganWP57vjEZUqK4k9VhuwrgcRGRbtBS2xTqNwy7tnZGvDJsJSGot2BekLFOl5s1N4Ykf0jHnKdrEwb6QbAwH2kYnmYVmO49zNZOJhvRDd34xYZIS5C0mNIhKxBNyi0wjN0nNuphmTTlx/vfHY4OUkjpS0ps6boq0o8pbnmhVA5AUHui3WA/U+aVliuV4DsbA+OMFtY9HFbHl80akOJhpMJWGtF0gZDaWwge0i5jfn5Rg6RPbyi0l5Q2PkHrWYpOsXLXWKLHjGRnRq4iB7OGzMej1cYWE9TQw4h8oizLHGI0QnIRIsowzaFkh5Ah8sQqS4sJJEFjMrKAimHiPdGIkVIcxSQtkix8uBTyhaILoJgFQMG7aY+EFhWgrGRofTiTmeROAxJCE5Mp9Ftl738xA+pmotOy2CLdqi1qdp76TI6x+tlno5gOZI81/zDHneGyNubMwgSmatzLPN6eyGuFGlscGStStTnwJoQdpqQdqZPN0YFa7Gw7OcJudjKguxBpOdgYF5gJMaukprZUOMVJMo7oWmxrGqRVWRHnkUjTuAZxiac0xKlDrMBzOfIbYDyHRUtk0CM+XMWauGNThxipMsVttYJlWaZ0e1mpLBHAzCoK8qxFLnfR1N4rVczeUqORUkHvhvp5Rxu5AM0XkVRDnRpAaTZ0xNRNmEeiidbHm11fzQWRzvyb6WWY8zaz0N45kDIcBjWkdBEbGK3HU+5h9Uqnp9hTDWaFhjGCoAitaTwgf0kp9aNy1PA/pGZF5+kLjW5h2nnFCbgYv2maIz50yMvMrNTEIs2FiurGuMehBrcZOsqvCHhYlK0hlBA1IhoVImEMVDsETLLg0GDaPBBEiwtyghAYcxyAyRHafMam4+6OfvIoxy7ifhHSTlAnpaygNSEfxbU4KS7DHR0m0N0brC8sXJhLKMmxLAbm2tzGPPONAayy6dZajetD3iuHKtYwLlNkRzpKnG6OYrXvBqOyMfF+JThtNWv3HJdNF7o2n01ZDjSnAX08FoIpWnWGzjIMeTNAtpDR84gmVaCv3ZirMXDjQP3sYHLfY9JAYGU4+4AD3NSNHH1OLJxJffYH4MkGVq1hs5ONnd/xT5wHaoenhGe2tYlkmRZ7PJb1llhn9o4++OdWq2WhTRyFPIH5xUmWp2FC5PLD3UhjRL3LKC9gx01rbMevTT2mH1cP0iijtECVut7zT1sF2L8TvMVKRpaP0LPmmqphvLKB76xOmONan+rCfAYeS6v0habjXujsYaAjyc6tmSpmTGF5sAFxoOZHCD26u6Nfp3WJHn+rjqyuiAtEbtE5lrxiCZLG/whhNC6xyM+1PGFb4ILTJWnneH9YyLTIHrnugeXPGtg+LBIHLTKMUJiDfBDM0ep9NvCK83RabXbw+UZWXPE0seJoxFUbI9GqLBJG1z2j5R6FfGiG0M//Z',
                          ),
                          fit: BoxFit.cover, // 이미지를 컨테이너 크기에 맞게 조정
                        ),
                      ),
                    ),
                    SizedBox(width: 16.0),
                    Container(
                      width: 100,
                      height: 100,
                      decoration: BoxDecoration(
                        color: Colors.white,
                        borderRadius: BorderRadius.circular(10),
                        border: Border.all(
                          color: Colors.green,
                          width: 2.0,
                        ),
                        image: DecorationImage(
                          image: NetworkImage(
                            'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQODZsm7jSf3mXAYFrBVfcKmMTUICGMnCGq9g&s', // 여기에 이미지 URL을 입력하세요
                          ),
                          fit: BoxFit.cover, // 이미지를 컨테이너 크기에 맞게 조정
                        ),
                      ),
                    ),
                    SizedBox(width: 16.0),
                    Container(
                      width: 100,
                      height: 100,
                      decoration: BoxDecoration(
                        color: Colors.white,
                        borderRadius: BorderRadius.circular(10),
                        border: Border.all(
                          color: Colors.green,
                          width: 2.0,
                        ),
                        image: DecorationImage(
                          image: NetworkImage(
                            'https://image.artbox.co.kr/upload/C00001/goods/800_800/388/240221005018388.jpg?s=/goods/org/388/240221005018388.jpg', // 여기에 이미지 URL을 입력하세요
                          ),
                          fit: BoxFit.cover, // 이미지를 컨테이너 크기에 맞게 조정
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),
            // 녹색 박스
            Container(
              margin: const EdgeInsets.symmetric(horizontal: 16.0),
              padding: const EdgeInsets.all(10.0),
              width: 350,
              height: 100,
              decoration: BoxDecoration(
                color: const Color(0xFF87b17c),
                borderRadius: BorderRadius.circular(21),
              ),
              child: RichText(
                  text: const TextSpan(
                      children: [
                        TextSpan(
                            text: "  오늘은 ",
                            style: TextStyle(color: Color(0xFF5b8868), fontWeight: FontWeight.w600, fontSize: 17)
                        ),
                        TextSpan(
                            text: "2024",
                            style: TextStyle(color: Color(0xffaed0a5), fontWeight: FontWeight.w900, fontSize: 17)
                        ),
                        TextSpan(
                            text: "년 ",
                            style: TextStyle(color: Color(0xFF5b8868), fontWeight: FontWeight.w600, fontSize: 17)
                        ),
                        TextSpan(
                            text: "10",
                            style: TextStyle(color: Color(0xffaed0a5), fontWeight: FontWeight.w900, fontSize: 17)
                        ),
                        TextSpan(
                            text: "월 " ,
                            style: TextStyle(color: Color(0xFF5b8868), fontWeight: FontWeight.w600, fontSize: 17)
                        ),
                        TextSpan(
                            text: "24",
                            style: TextStyle(color: Color(0xffaed0a5), fontWeight: FontWeight.w900, fontSize: 17)
                        ),
                        TextSpan(
                            text: "일 입니다.",
                            style: TextStyle(color: Color(0xFF5b8868), fontWeight: FontWeight.w600, fontSize: 17)
                        ),
                      ]
                  )
              ),
            ),
            const SizedBox(height: 16.0),
            // 리스트 아이템들
            Container(
              margin: const EdgeInsets.symmetric(vertical: 8.0, horizontal: 16.0),
              padding: const EdgeInsets.all(8.0),
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(16),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.1),
                    blurRadius: 5,
                  ),
                ],
              ),
              child: ExpansionTile(
                title: const Text(
                  '새싹이',
                  style: TextStyle(),
                ),

                children: <Widget>[
                  Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.start,
                      children: [
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "물주기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "08:00 AM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked0Extra[0],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked0Extra[0] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                        SizedBox(height: 10),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "햇빛 쪼여주기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "13:00 PM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked0Extra[1],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked0Extra[1] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                        SizedBox(height: 10),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "분갈이하기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "17:00 PM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked0Extra[2],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked0Extra[2] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
// 다른 항목
            Container(
              margin: const EdgeInsets.symmetric(vertical: 8.0, horizontal: 16.0),
              padding: const EdgeInsets.all(8.0),
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(16),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.1),
                    blurRadius: 5,
                  ),
                ],
              ),
              child: ExpansionTile(
                title: const Text(
                  '튼튼이',
                  style: TextStyle(),
                ),

                children: <Widget>[
                  Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.start,
                      children: [
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "물주기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "08:00 AM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked1Extra[0],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked1Extra[0] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                        SizedBox(height: 10),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "햇빛 쪼여주기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "13:00 PM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked1Extra[1],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked1Extra[1] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                        SizedBox(height: 10),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "분갈이하기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "17:00 PM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked1Extra[2],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked1Extra[2] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
// 또 다른 항목
            Container(
              margin: const EdgeInsets.symmetric(vertical: 8.0, horizontal: 16.0),
              padding: const EdgeInsets.all(8.0),
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(16),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.1),
                    blurRadius: 5,
                  ),
                ],
              ),
              child: ExpansionTile(
                title: const Text(
                  '서현이',
                  style: TextStyle(),
                ),

                children: <Widget>[
                  Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.start,
                      children: [
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "물주기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "08:00 AM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked2Extra[0],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked2Extra[0] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                        SizedBox(height: 10),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "햇빛 쪼여주기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "13:00 PM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked2Extra[1],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked2Extra[1] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                        SizedBox(height: 10),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                                child: RichText(
                                    text: const TextSpan(
                                        children: [
                                          TextSpan(
                                              text: "분갈이하기 \n",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w900)
                                          ),
                                          TextSpan(
                                              text: "17:00 PM",
                                              style: TextStyle(color: Color(0xff616161), fontWeight: FontWeight.w100)
                                          )
                                        ]
                                    )

                                )


                            ),
                            Checkbox(
                              value: _checked2Extra[2],
                              onChanged: (bool? value) {
                                setState(() {
                                  _checked2Extra[2] = value!;
                                });
                              },
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class ProfileScreen extends StatefulWidget {
  @override
  _ProfileScreenState createState() => _ProfileScreenState();
}

class _ProfileScreenState extends State<ProfileScreen> {
  final TextEditingController _idController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  bool _isPasswordVisible = true;  // 초기 상태: 비밀번호가 보임

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,  // 배경색 흰색으로 설정
      appBar: AppBar(
        backgroundColor: Colors.white,
        elevation: 0,
        centerTitle: true,
        title: Padding(
          padding: const EdgeInsets.only(top: 80.0),  // ZARAM 로고를 좀 더 아래로 배치
          child: Text(
            'ZARAM',
            style: TextStyle(
              fontSize: 40,  // ZARAM 로고 크기 키움
              fontWeight: FontWeight.bold,  // 로고를 더 두껍게
              color: Color(0xFF92C377), // 로고 색상
            ),
          ),
        ),
        toolbarHeight: 200, // AppBar 높이 조정
      ),
      body: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 24.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            // 이메일 입력
            TextField(
              controller: _idController,
              decoration: InputDecoration(
                labelText: '이메일 주소 또는 아이디',
                border: OutlineInputBorder(),
              ),
            ),
            SizedBox(height: 16),

            // 비밀번호 입력
            TextField(
              controller: _passwordController,
              decoration: InputDecoration(
                labelText: '비밀번호',
                border: OutlineInputBorder(),
                suffixIcon: IconButton(
                  icon: Icon(
                    _isPasswordVisible ? Icons.visibility : Icons.visibility_off,
                  ),
                  onPressed: () {
                    setState(() {
                      _isPasswordVisible = !_isPasswordVisible;
                    });
                  },
                ),
              ),
              obscureText: _isPasswordVisible,  // 가시성 상태에 따라 비밀번호 숨김 여부 결정
            ),
            SizedBox(height: 16),

            // 로그인 상태 유지 체크박스
            Row(
              children: [
                Checkbox(value: true, onChanged: (value) {}),
                Text('로그인 상태 유지'),
              ],
            ),
            SizedBox(height: 16),

            // 로그인 버튼
            ElevatedButton(
              onPressed: _login,
              style: ElevatedButton.styleFrom(
                backgroundColor: Color(0xFF92C377), // 버튼 배경색
                padding: EdgeInsets.symmetric(vertical: 14), // 버튼 패딩
              ),
              child: Text('로그인하기', style: TextStyle(fontSize: 18)),
            ),
            SizedBox(height: 20),

            // 회원가입 링크 추가
            TextButton(
              onPressed: () {
                Navigator.pushNamed(context, '/signup'); // 회원가입 페이지로 이동
              },
              child: Text('아이디찾기 | 비밀번호 찾기 | 회원가입',
                  style: TextStyle(color: Colors.black45)),
            ),
            SizedBox(height: 20),

            // SNS 로그인으로 간편하게
            Center(
              child: Text(
                'SNS 계정으로 로그인하기',
                style: TextStyle(fontSize: 16, color: Colors.black45),
              ),
            ),
            SizedBox(height: 20),

            // 소셜 로그인 버튼들 (가로로 배치)
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,  // 버튼들 가로로 균등 배치
              children: [
                ElevatedButton(
                    onPressed: () {
                      // 카카오톡 로그인 로직
                    },
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.transparent, // 배경색 제거
                      elevation: 0, // 그림자 제거
                      padding: EdgeInsets.zero, // 패딩 제거
                    ),
                    child: Image.asset(
                      'assets/카카오톡.png',
                      height: 73,
                      width: 73,
                    )
                ),
                ElevatedButton(
                    onPressed: () {
                      // 카카오톡 로그인 로직
                    },
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.transparent, // 배경색 제거
                      elevation: 0, // 그림자 제거
                      padding: EdgeInsets.zero, // 패딩 제거
                    ),
                    child: Image.asset(
                      'assets/구글.png',
                      height: 70,
                      width: 70,
                    )
                ),
                ElevatedButton(
                    onPressed: () {

                    },
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.transparent, // 배경색 제거
                      elevation: 0, // 그림자 제거
                      padding: EdgeInsets.zero, // 패딩 제거
                    ),
                    child: Image.asset(
                      'assets/네이버.png',
                      height: 50,
                      width: 50,
                    )
                ),
                ElevatedButton(
                    onPressed: () {

                    },
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.transparent, // 배경색 제거
                      elevation: 0, // 그림자 제거
                      padding: EdgeInsets.zero, // 패딩 제거
                    ),
                    child: Image.asset(
                      'assets/애플.png',
                      height: 40,
                      width: 40,
                    )
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  // 소셜 로그인 버튼 (원 안에 아이콘)



  void _login() {
    // 로그인 로직
  }
}
// 회원가입 페이지 구현
class SignupScreen extends StatelessWidget {
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('회원가입'),
        backgroundColor: Color(0xFF92C377), // AppBar 색상
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            TextField(
              controller: _usernameController,
              decoration: InputDecoration(labelText: '사용자 이름'),
            ),
            SizedBox(height: 16),
            TextField(
              controller: _emailController,
              decoration: InputDecoration(labelText: '이메일'),
            ),
            SizedBox(height: 16),
            TextField(
              controller: _passwordController,
              decoration: InputDecoration(labelText: '비밀번호'),
              obscureText: true,
            ),
            SizedBox(height: 24),
            ElevatedButton(
              onPressed: () {
                // 회원가입 로직 구현
                Navigator.pop(context); // 회원가입 후 로그인 페이지로 이동
              },
              child: Text('회원가입하기'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Color(0xFF92C377), // 회원가입 버튼 색상
                padding: EdgeInsets.symmetric(vertical: 14),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
