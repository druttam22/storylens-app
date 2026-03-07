import React, { useState, useRef, useEffect } from 'react';
import {
  View, Text, TouchableOpacity, ScrollView, TextInput,
  StyleSheet, Image, Alert, FlatList, Modal,
  StatusBar, Dimensions, SafeAreaView
} from 'react-native';
import { Camera } from 'expo-camera';
import * as MediaLibrary from 'expo-media-library';
import * as ImageManipulator from 'expo-image-manipulator';
import AsyncStorage from '@react-native-async-storage/async-storage';

const { width } = Dimensions.get('window');

const DEFAULT_CATEGORIES = [
  { id: 'daughter', label: 'My Daughter', emoji: '👧', color: '#FF6B9D', bg: '#FFF0F6' },
  { id: 'farm',     label: 'Farm',        emoji: '🌾', color: '#52B788', bg: '#F0FFF4' },
  { id: 'house',    label: 'House Build', emoji: '🏗️', color: '#F4A261', bg: '#FFF8F0' },
  { id: 'garden',   label: 'Garden',      emoji: '🌻', color: '#A8C686', bg: '#F4FFF0' },
  { id: 'pet',      label: 'Pet Growth',  emoji: '🐾', color: '#9B72CF', bg: '#F8F0FF' },
  { id: 'fitness',  label: 'Fitness',     emoji: '💪', color: '#E76F51', bg: '#FFF4F0' },
];

const QUALITY_OPTIONS = [
  { label: 'Minimum (~100KB)', value: 0.3, desc: '480p — saves most storage' },
  { label: 'Medium (~300KB)',  value: 0.6, desc: '720p — good balance' },
  { label: 'High (~700KB)',    value: 0.85, desc: '1080p — best quality' },
];

function formatDate(ts) {
  const d = new Date(ts);
  return d.toLocaleDateString('en-IN', { day: '2-digit', month: 'short', year: 'numeric' });
}

function formatTime(ts) {
  return new Date(ts).toLocaleTimeString('en-IN', { hour: '2-digit', minute: '2-digit' });
}

export default function App() {
  const [screen, setScreen]               = useState('home');
  const [categories, setCategories]       = useState(DEFAULT_CATEGORIES);
  const [stories, setStories]             = useState({});
  const [activeCategory, setActiveCategory] = useState(null);
  const [quality, setQuality]             = useState(0.3);
  const [hasCamPerm, setHasCamPerm]       = useState(false);
  const [cameraReady, setCameraReady]     = useState(false);
  const [capturedUri, setCapturedUri]     = useState(null);
  const [note, setNote]                   = useState('');
  const [newCatName, setNewCatName]       = useState('');
  const [newCatEmoji, setNewCatEmoji]     = useState('📷');
  const [viewPhoto, setViewPhoto]         = useState(null);
  const [lightbox, setLightbox]           = useState(null);
  const [cameraType, setCameraType]       = useState('back');
  const cameraRef = useRef(null);

  const cat = categories.find(c => c.id === activeCategory);
  const photos = activeCategory ? (stories[activeCategory] || []) : [];
  const sortedPhotos = [...photos].sort((a, b) => b.ts - a.ts);

  useEffect(() => {
    loadData();
    requestPermissions();
  }, []);

  const requestPermissions = async () => {
    const { status } = await Camera.requestCameraPermissionsAsync();
    await MediaLibrary.requestPermissionsAsync();
    setHasCamPerm(status === 'granted');
  };

  const loadData = async () => {
    try {
      const cats = await AsyncStorage.getItem('categories');
      const strs = await AsyncStorage.getItem('stories');
      const qual = await AsyncStorage.getItem('quality');
      if (cats) setCategories(JSON.parse(cats));
      if (strs) setStories(JSON.parse(strs));
      if (qual) setQuality(parseFloat(qual));
    } catch (e) { console.log(e); }
  };

  const saveData = async (newCats, newStories) => {
    try {
      await AsyncStorage.setItem('categories', JSON.stringify(newCats || categories));
      await AsyncStorage.setItem('stories', JSON.stringify(newStories || stories));
    } catch (e) { console.log(e); }
  };

  const takePicture = async () => {
    if (!cameraRef.current || !cameraReady) return;
    try {
      const photo = await cameraRef.current.takePictureAsync({ quality: 1 });
      const maxW = quality < 0.5 ? 480 : quality < 0.7 ? 720 : 1080;
      const compressed = await ImageManipulator.manipulateAsync(
        photo.uri,
        [{ resize: { width: maxW } }],
        { compress: quality, format: ImageManipulator.SaveFormat.JPEG }
      );
      setCapturedUri(compressed.uri);
      setScreen('preview');
    } catch (e) { Alert.alert('Error', 'Could not take photo'); }
  };

  const savePhoto = async () => {
    if (!capturedUri || !activeCategory) return;
    const entry = { id: Date.now(), ts: Date.now(), note, uri: capturedUri };
    const updated = { ...stories, [activeCategory]: [entry, ...(stories[activeCategory] || [])] };
    setStories(updated);
    await saveData(null, updated);
    setCapturedUri(null);
    setNote('');
    setScreen('story');
  };

  const addCategory = async () => {
    if (!newCatName.trim()) return;
    const colors = ['#E63946','#2A9D8F','#E9C46A','#F4A261','#264653','#6A4C93','#1982C4'];
    const color = colors[Math.floor(Math.random() * colors.length)];
    const id = 'cat_' + Date.now();
    const newCat = { id, label: newCatName.trim(), emoji: newCatEmoji, color, bg: '#F9F9F9' };
    const updated = [...categories, newCat];
    setCategories(updated);
    setNewCatName(''); setNewCatEmoji('📷');
    await saveData(updated, null);
    setScreen('home');
  };

  // ── HOME ─────────────────────────────────────────────────────────────────
  if (screen === 'home') return (
    <SafeAreaView style={s.safe}>
      <StatusBar barStyle="dark-content" backgroundColor="#fff" />
      <View style={s.homeHeader}>
        <View>
          <Text style={s.appName}>StoryLens</Text>
          <Text style={s.appSub}>Your daily visual diary</Text>
        </View>
        <TouchableOpacity onPress={() => setScreen('settings')} style={s.settingsBtn}>
          <Text style={{ fontSize: 22 }}>⚙️</Text>
        </TouchableOpacity>
      </View>

      <View style={s.statsRow}>
        {[
          { label: 'Stories', val: categories.length },
          { label: 'Photos', val: Object.values(stories).flat().length },
        ].map(st => (
          <View key={st.label} style={s.statBox}>
            <Text style={s.statNum}>{st.val}</Text>
            <Text style={s.statLabel}>{st.label}</Text>
          </View>
        ))}
      </View>

      <Text style={s.sectionTitle}>My Stories</Text>
      <ScrollView contentContainerStyle={s.grid}>
        {categories.map(c => (
          <TouchableOpacity key={c.id}
            style={[s.catCard, { backgroundColor: c.bg, borderColor: c.color + '55' }]}
            onPress={() => { setActiveCategory(c.id); setScreen('story'); }}>
            <View style={[s.catEmoji, { backgroundColor: c.color + '22' }]}>
              <Text style={{ fontSize: 26 }}>{c.emoji}</Text>
            </View>
            <Text style={s.catLabel}>{c.label}</Text>
            <Text style={[s.catCount, { color: c.color }]}>
              {(stories[c.id] || []).length} photos
            </Text>
          </TouchableOpacity>
        ))}
        <TouchableOpacity style={[s.catCard, { backgroundColor: '#F0F6FF', borderColor: '#6C9BD244' }]}
          onPress={() => setScreen('newstory')}>
          <View style={[s.catEmoji, { backgroundColor: '#6C9BD222' }]}>
            <Text style={{ fontSize: 26 }}>✨</Text>
          </View>
          <Text style={s.catLabel}>New Story</Text>
        </TouchableOpacity>
      </ScrollView>
    </SafeAreaView>
  );

  // ── NEW STORY ─────────────────────────────────────────────────────────────
  if (screen === 'newstory') return (
    <SafeAreaView style={s.safe}>
      <View style={s.topBar}>
        <TouchableOpacity onPress={() => setScreen('home')}><Text style={s.backBtn}>← Back</Text></TouchableOpacity>
        <Text style={s.topTitle}>New Story</Text>
        <View style={{ width: 60 }} />
      </View>
      <ScrollView contentContainerStyle={{ padding: 20 }}>
        <Text style={{ fontSize: 64, textAlign: 'center', marginBottom: 10 }}>{newCatEmoji}</Text>
        <View style={s.emojiRow}>
          {['📷','🌱','🏠','👶','🐕','🚗','🍳','✈️','🎨','💡','🌊','🏋️'].map(e => (
            <TouchableOpacity key={e}
              style={[s.emojiOpt, { backgroundColor: newCatEmoji === e ? '#E8F4FF' : '#F5F5F5' }]}
              onPress={() => setNewCatEmoji(e)}>
              <Text style={{ fontSize: 22 }}>{e}</Text>
            </TouchableOpacity>
          ))}
        </View>
        <TextInput
          placeholder="Story name (e.g. My Farm)"
          value={newCatName}
          onChangeText={setNewCatName}
          style={s.input}
        />
        <TouchableOpacity onPress={addCategory} style={s.primaryBtn}>
          <Text style={s.primaryBtnText}>✨ Create Story</Text>
        </TouchableOpacity>
      </ScrollView>
    </SafeAreaView>
  );

  // ── STORY ─────────────────────────────────────────────────────────────────
  if (screen === 'story' && cat) return (
    <SafeAreaView style={s.safe}>
      <View style={[s.storyHeader, { backgroundColor: cat.color }]}>
        <View style={s.topBar2}>
          <TouchableOpacity onPress={() => setScreen('home')}><Text style={s.backBtnW}>← Back</Text></TouchableOpacity>
          <Text style={s.topTitleW}>{cat.emoji} {cat.label}</Text>
          <View style={{ width: 60 }} />
        </View>
        <Text style={s.storyMeta}>{photos.length} photos</Text>
      </View>

      <View style={s.actionRow}>
        <TouchableOpacity style={[s.actionBtn, { backgroundColor: cat.color }]}
          onPress={() => { setCapturedUri(null); setScreen('camera'); }}>
          <Text style={s.actionBtnText}>📸 Take Photo</Text>
        </TouchableOpacity>
        <TouchableOpacity style={[s.actionBtn, { backgroundColor: '#555' }]}
          onPress={() => setScreen('timeline')}>
          <Text style={s.actionBtnText}>📅 Timeline</Text>
        </TouchableOpacity>
        <TouchableOpacity style={[s.actionBtn, { backgroundColor: '#555' }]}
          onPress={() => setScreen('gallery')}>
          <Text style={s.actionBtnText}>🖼 Gallery</Text>
        </TouchableOpacity>
      </View>

      <Text style={s.sectionTitle}>Recent Photos</Text>
      {sortedPhotos.length === 0 && (
        <View style={s.emptyState}>
          <Text style={{ fontSize: 48 }}>{cat.emoji}</Text>
          <Text style={s.emptyText}>No photos yet!</Text>
          <Text style={s.emptySub}>Tap "Take Photo" to start your story</Text>
        </View>
      )}
      <FlatList
        data={sortedPhotos.slice(0, 10)}
        keyExtractor={p => p.id.toString()}
        contentContainerStyle={{ padding: 16, paddingBottom: 80 }}
        renderItem={({ item: p }) => (
          <TouchableOpacity style={s.recentCard} onPress={() => setViewPhoto(p)}>
            <View style={[s.photoThumb, { backgroundColor: cat.color + '22' }]}>
              {p.uri
                ? <Image source={{ uri: p.uri }} style={s.thumbImg} />
                : <Text style={{ fontSize: 28 }}>{cat.emoji}</Text>}
            </View>
            <View style={{ flex: 1 }}>
              <Text style={s.recentDate}>{formatDate(p.ts)}</Text>
              <Text style={s.recentTime}>{formatTime(p.ts)}</Text>
              {p.note ? <Text style={s.recentNote}>"{p.note}"</Text> : null}
            </View>
          </TouchableOpacity>
        )}
      />

      <Modal visible={!!viewPhoto} transparent animationType="fade">
        <TouchableOpacity style={s.overlay} activeOpacity={1} onPress={() => setViewPhoto(null)}>
          <View style={s.viewCard} onStartShouldSetResponder={() => true}>
            {viewPhoto?.uri && <Image source={{ uri: viewPhoto.uri }} style={s.viewImg} resizeMode="contain" />}
            <Text style={s.viewDate}>{viewPhoto ? formatDate(viewPhoto.ts) : ''}</Text>
            {viewPhoto?.note ? <Text style={s.viewNote}>{viewPhoto.note}</Text> : null}
            <TouchableOpacity onPress={() => setViewPhoto(null)} style={s.closeBtn}>
              <Text style={{ fontWeight: '700', fontSize: 15 }}>Close</Text>
            </TouchableOpacity>
          </View>
        </TouchableOpacity>
      </Modal>
    </SafeAreaView>
  );

  // ── CAMERA ────────────────────────────────────────────────────────────────
  if (screen === 'camera') return (
    <View style={{ flex: 1, backgroundColor: '#000' }}>
      {!hasCamPerm ? (
        <View style={s.permWrap}>
          <Text style={{ color: '#fff', fontSize: 16, textAlign: 'center', marginBottom: 20 }}>
            Camera permission is required
          </Text>
          <TouchableOpacity onPress={requestPermissions} style={s.primaryBtn}>
            <Text style={s.primaryBtnText}>Grant Permission</Text>
          </TouchableOpacity>
          <TouchableOpacity onPress={() => setScreen('story')} style={{ marginTop: 12 }}>
            <Text style={{ color: '#aaa', textAlign: 'center' }}>← Go Back</Text>
          </TouchableOpacity>
        </View>
      ) : (
        <>
          <Camera
            ref={cameraRef}
            style={{ flex: 1 }}
            type={cameraType}
            onCameraReady={() => setCameraReady(true)}
          />
          <View style={s.camControls}>
            <TouchableOpacity onPress={() => setScreen('story')} style={s.camBackBtn}>
              <Text style={{ color: '#fff', fontSize: 15, fontWeight: '700' }}>← Back</Text>
            </TouchableOpacity>
            <TouchableOpacity onPress={takePicture} style={s.snapBtn} />
            <TouchableOpacity
              onPress={() => setCameraType(cameraType === 'back' ? 'front' : 'back')}
              style={s.flipBtn}>
              <Text style={{ fontSize: 28 }}>🔄</Text>
            </TouchableOpacity>
          </View>
        </>
      )}
    </View>
  );

  // ── PREVIEW ───────────────────────────────────────────────────────────────
  if (screen === 'preview') return (
    <SafeAreaView style={[s.safe, { backgroundColor: '#111' }]}>
      <View style={[s.topBar, { backgroundColor: '#111' }]}>
        <TouchableOpacity onPress={() => { setCapturedUri(null); setScreen('camera'); }}>
          <Text style={[s.backBtn, { color: '#fff' }]}>← Retake</Text>
        </TouchableOpacity>
        <Text style={[s.topTitle, { color: '#fff' }]}>Preview</Text>
        <View style={{ width: 60 }} />
      </View>
      {capturedUri && <Image source={{ uri: capturedUri }} style={s.previewImg} resizeMode="cover" />}
      <View style={{ padding: 20 }}>
        <TextInput
          placeholder="Add a note (optional)..."
          placeholderTextColor="#888"
          value={note}
          onChangeText={setNote}
          style={[s.input, { backgroundColor: '#222', color: '#fff', borderColor: '#333' }]}
        />
        <TouchableOpacity onPress={savePhoto} style={[s.primaryBtn, { backgroundColor: '#4A90E2', marginTop: 8 }]}>
          <Text style={s.primaryBtnText}>💾 Save to Story</Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  );

  // ── TIMELINE ──────────────────────────────────────────────────────────────
  if (screen === 'timeline' && cat) {
    const sorted = [...photos].sort((a, b) => a.ts - b.ts);
    return (
      <SafeAreaView style={s.safe}>
        <View style={[s.storyHeader, { backgroundColor: cat.color }]}>
          <View style={s.topBar2}>
            <TouchableOpacity onPress={() => setScreen('story')}><Text style={s.backBtnW}>← Back</Text></TouchableOpacity>
            <Text style={s.topTitleW}>📅 Timeline</Text>
            <View style={{ width: 60 }} />
          </View>
          <Text style={s.storyMeta}>{cat.emoji} {cat.label} · {photos.length} entries</Text>
        </View>
        <ScrollView contentContainerStyle={{ padding: 16, paddingBottom: 80 }}>
          {sorted.length === 0 && (
            <View style={s.emptyState}>
              <Text style={{ fontSize: 40 }}>📅</Text>
              <Text style={s.emptyText}>No entries yet</Text>
            </View>
          )}
          {sorted.map((p, i) => (
            <View key={p.id} style={{ flexDirection: 'row', gap: 12, marginBottom: 16 }}>
              <View style={{ alignItems: 'center', width: 20 }}>
                <View style={[s.dot, { backgroundColor: cat.color }]} />
                {i < sorted.length - 1 && <View style={[s.vline, { backgroundColor: cat.color + '44' }]} />}
              </View>
              <TouchableOpacity style={s.tlCard} onPress={() => setViewPhoto(p)}>
                {p.uri
                  ? <Image source={{ uri: p.uri }} style={s.tlThumb} />
                  : <View style={[s.tlThumb, { backgroundColor: cat.color + '22', alignItems: 'center', justifyContent: 'center' }]}>
                      <Text style={{ fontSize: 24 }}>{cat.emoji}</Text>
                    </View>}
                <View style={{ flex: 1, padding: 10 }}>
                  <Text style={s.recentDate}>{formatDate(p.ts)}</Text>
                  {p.note ? <Text style={s.recentNote}>{p.note}</Text> : null}
                </View>
              </TouchableOpacity>
            </View>
          ))}
        </ScrollView>
        <Modal visible={!!viewPhoto} transparent animationType="fade">
          <TouchableOpacity style={s.overlay} activeOpacity={1} onPress={() => setViewPhoto(null)}>
            <View style={s.viewCard} onStartShouldSetResponder={() => true}>
              {viewPhoto?.uri && <Image source={{ uri: viewPhoto.uri }} style={s.viewImg} resizeMode="contain" />}
              <Text style={s.viewDate}>{viewPhoto ? formatDate(viewPhoto.ts) : ''}</Text>
              {viewPhoto?.note ? <Text style={s.viewNote}>{viewPhoto.note}</Text> : null}
              <TouchableOpacity onPress={() => setViewPhoto(null)} style={s.closeBtn}>
                <Text style={{ fontWeight: '700', fontSize: 15 }}>Close</Text>
              </TouchableOpacity>
            </View>
          </TouchableOpacity>
        </Modal>
      </SafeAreaView>
    );
  }

  // ── GALLERY ───────────────────────────────────────────────────────────────
  if (screen === 'gallery' && cat) {
    const cellSize = (width - 40) / 3;
    return (
      <SafeAreaView style={s.safe}>
        <View style={[s.storyHeader, { backgroundColor: cat.color }]}>
          <View style={s.topBar2}>
            <TouchableOpacity onPress={() => setScreen('story')}><Text style={s.backBtnW}>← Back</Text></TouchableOpacity>
            <Text style={s.topTitleW}>🖼 Gallery</Text>
            <View style={{ width: 60 }} />
          </View>
          <Text style={s.storyMeta}>{cat.emoji} {cat.label} · {photos.length} photos</Text>
        </View>
        {sortedPhotos.length === 0 && (
          <View style={s.emptyState}>
            <Text style={{ fontSize: 40 }}>🖼</Text>
            <Text style={s.emptyText}>No photos yet</Text>
          </View>
        )}
        <FlatList
          data={sortedPhotos}
          keyExtractor={p => p.id.toString()}
          numColumns={3}
          contentContainerStyle={{ padding: 8, paddingBottom: 80 }}
          columnWrapperStyle={{ gap: 4, marginBottom: 4 }}
          renderItem={({ item: p }) => (
            <TouchableOpacity onPress={() => setViewPhoto(p)}
              style={{ width: cellSize, height: cellSize, borderRadius: 8, overflow: 'hidden', backgroundColor: cat.color + '22' }}>
              {p.uri
                ? <Image source={{ uri: p.uri }} style={{ width: '100%', height: '100%' }} />
                : <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
                    <Text style={{ fontSize: 28 }}>{cat.emoji}</Text>
                  </View>}
              <View style={s.galDateBadge}>
                <Text style={{ color: '#fff', fontSize: 8, fontWeight: '700' }}>{formatDate(p.ts)}</Text>
              </View>
            </TouchableOpacity>
          )}
        />
        <Modal visible={!!viewPhoto} transparent animationType="fade">
          <TouchableOpacity style={s.overlay} activeOpacity={1} onPress={() => setViewPhoto(null)}>
            <View style={s.viewCard} onStartShouldSetResponder={() => true}>
              {viewPhoto?.uri && <Image source={{ uri: viewPhoto.uri }} style={s.viewImg} resizeMode="contain" />}
              <Text style={s.viewDate}>{viewPhoto ? formatDate(viewPhoto.ts) : ''}</Text>
              {viewPhoto?.note ? <Text style={s.viewNote}>{viewPhoto.note}</Text> : null}
              <TouchableOpacity onPress={() => setViewPhoto(null)} style={s.closeBtn}>
                <Text style={{ fontWeight: '700', fontSize: 15 }}>Close</Text>
              </TouchableOpacity>
            </View>
          </TouchableOpacity>
        </Modal>
      </SafeAreaView>
    );
  }

  // ── SETTINGS ──────────────────────────────────────────────────────────────
  if (screen === 'settings') return (
    <SafeAreaView style={s.safe}>
      <View style={s.topBar}>
        <TouchableOpacity onPress={() => setScreen('home')}><Text style={s.backBtn}>← Back</Text></TouchableOpacity>
        <Text style={s.topTitle}>⚙️ Settings</Text>
        <View style={{ width: 60 }} />
      </View>
      <ScrollView contentContainerStyle={{ padding: 20, paddingBottom: 80 }}>
        <Text style={s.sectionTitle}>📦 Default Photo Quality</Text>
        <Text style={{ color: '#888', fontSize: 13, marginBottom: 12 }}>
          Lower quality = less storage used on your phone
        </Text>
        {QUALITY_OPTIONS.map(q => (
          <TouchableOpacity key={q.value}
            onPress={async () => { setQuality(q.value); await AsyncStorage.setItem('quality', q.value.toString()); }}
            style={[s.qualOpt, { borderColor: quality === q.value ? '#4A90E2' : '#EEE', backgroundColor: quality === q.value ? '#EEF6FF' : '#fff' }]}>
            <Text style={{ fontWeight: '700', fontSize: 14 }}>{q.label}</Text>
            <Text style={{ color: '#888', fontSize: 12, marginTop: 2 }}>{q.desc}</Text>
          </TouchableOpacity>
        ))}

        <Text style={[s.sectionTitle, { marginTop: 24 }]}>📊 Storage Summary</Text>
        {categories.map(c => (
          <View key={c.id} style={s.storageRow}>
            <Text>{c.emoji} {c.label}</Text>
            <Text style={{ color: '#888' }}>{(stories[c.id] || []).length} photos</Text>
          </View>
        ))}
        <View style={[s.storageRow, { borderTopWidth: 1, borderTopColor: '#EEE', marginTop: 8, paddingTop: 8 }]}>
          <Text style={{ fontWeight: '800' }}>Total</Text>
          <Text style={{ fontWeight: '800' }}>{Object.values(stories).flat().length} photos</Text>
        </View>
      </ScrollView>
    </SafeAreaView>
  );

  return null;
}

const s = StyleSheet.create({
  safe: { flex: 1, backgroundColor: '#FAFAFA' },
  homeHeader: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', padding: 16, backgroundColor: '#fff', borderBottomWidth: 1, borderBottomColor: '#F0F0F0' },
  appName: { fontSize: 26, fontWeight: '900', color: '#1A1A2E' },
  appSub: { fontSize: 12, color: '#999' },
  settingsBtn: { padding: 8 },
  statsRow: { flexDirection: 'row', gap: 8, padding: 12, backgroundColor: '#fff', borderBottomWidth: 1, borderBottomColor: '#F0F0F0' },
  statBox: { flex: 1, alignItems: 'center', padding: 8, backgroundColor: '#F7F8FF', borderRadius: 10 },
  statNum: { fontSize: 22, fontWeight: '800', color: '#1A1A2E' },
  statLabel: { fontSize: 10, color: '#888', marginTop: 2 },
  sectionTitle: { fontSize: 14, fontWeight: '800', color: '#1A1A2E', padding: 14, paddingBottom: 6 },
  grid: { flexDirection: 'row', flexWrap: 'wrap', padding: 10, gap: 10, paddingBottom: 80 },
  catCard: { width: (width - 40) / 2, borderRadius: 16, padding: 14, borderWidth: 1.5, minHeight: 90 },
  catEmoji: { width: 44, height: 44, borderRadius: 12, alignItems: 'center', justifyContent: 'center', marginBottom: 8 },
  catLabel: { fontSize: 13, fontWeight: '800', color: '#1A1A2E' },
  catCount: { fontSize: 11, marginTop: 2, fontWeight: '600' },
  topBar: { flexDirection: 'row', alignItems: 'center', justifyContent: 'space-between', padding: 14, backgroundColor: '#fff', borderBottomWidth: 1, borderBottomColor: '#EEE' },
  topTitle: { fontSize: 16, fontWeight: '800', color: '#1A1A2E' },
  backBtn: { fontSize: 14, color: '#4A90E2', fontWeight: '600' },
  emojiRow: { flexDirection: 'row', flexWrap: 'wrap', gap: 8, justifyContent: 'center', marginBottom: 16 },
  emojiOpt: { padding: 8, borderRadius: 10 },
  input: { padding: 14, fontSize: 15, borderWidth: 2, borderColor: '#EEE', borderRadius: 12, fontFamily: 'System', marginBottom: 12 },
  primaryBtn: { padding: 15, backgroundColor: '#1A1A2E', borderRadius: 14, alignItems: 'center' },
  primaryBtnText: { color: '#fff', fontSize: 15, fontWeight: '700' },
  storyHeader: { paddingBottom: 14 },
  topBar2: { flexDirection: 'row', alignItems: 'center', justifyContent: 'space-between', padding: 14 },
  topTitleW: { fontSize: 16, fontWeight: '800', color: '#fff' },
  backBtnW: { fontSize: 14, color: 'rgba(255,255,255,0.9)', fontWeight: '600' },
  storyMeta: { fontSize: 12, color: 'rgba(255,255,255,0.85)', fontWeight: '600', paddingHorizontal: 16 },
  actionRow: { flexDirection: 'row', gap: 8, padding: 12, backgroundColor: '#fff', borderBottomWidth: 1, borderBottomColor: '#F0F0F0' },
  actionBtn: { flex: 1, padding: 10, borderRadius: 10, alignItems: 'center' },
  actionBtnText: { color: '#fff', fontSize: 11, fontWeight: '700' },
  recentCard: { flexDirection: 'row', gap: 12, alignItems: 'center', backgroundColor: '#fff', borderRadius: 14, padding: 12, marginBottom: 10, elevation: 2 },
  photoThumb: { width: 60, height: 60, borderRadius: 12, alignItems: 'center', justifyContent: 'center', overflow: 'hidden' },
  thumbImg: { width: '100%', height: '100%' },
  recentDate: { fontSize: 14, fontWeight: '700', color: '#1A1A2E' },
  recentTime: { fontSize: 11, color: '#AAA' },
  recentNote: { fontSize: 12, color: '#666', marginTop: 3, fontStyle: 'italic' },
  emptyState: { alignItems: 'center', padding: 40 },
  emptyText: { fontSize: 16, fontWeight: '700', marginTop: 8, color: '#BBB' },
  emptySub: { fontSize: 12, color: '#CCC', marginTop: 4 },
  overlay: { flex: 1, backgroundColor: 'rgba(0,0,0,0.8)', alignItems: 'center', justifyContent: 'center', padding: 20 },
  viewCard: { backgroundColor: '#fff', borderRadius: 20, padding: 20, width: '100%', alignItems: 'center', gap: 10 },
  viewImg: { width: '100%', height: 280, borderRadius: 12 },
  viewDate: { fontSize: 13, color: '#888' },
  viewNote: { fontSize: 14, color: '#333', fontStyle: 'italic', textAlign: 'center' },
  closeBtn: { padding: 12, backgroundColor: '#F0F0F0', borderRadius: 10, width: '100%', alignItems: 'center' },
  permWrap: { flex: 1, alignItems: 'center', justifyContent: 'center', padding: 40 },
  camControls: { position: 'absolute', bottom: 40, left: 0, right: 0, flexDirection: 'row', justifyContent: 'space-around', alignItems: 'center', paddingHorizontal: 30 },
  camBackBtn: { padding: 12 },
  snapBtn: { width: 72, height: 72, borderRadius: 36, backgroundColor: '#fff', borderWidth: 4, borderColor: 'rgba(255,255,255,0.5)' },
  flipBtn: { padding: 12 },
  previewImg: { width: '100%', height: 340 },
  dot: { width: 12, height: 12, borderRadius: 6 },
  vline: { width: 2, flex: 1, marginTop: 4, minHeight: 24 },
  tlCard: { flex: 1, backgroundColor: '#fff', borderRadius: 14, flexDirection: 'row', overflow: 'hidden', elevation: 2 },
  tlThumb: { width: 72, height: 72 },
  galDateBadge: { position: 'absolute', bottom: 0, left: 0, right: 0, backgroundColor: 'rgba(0,0,0,0.5)', padding: 3, alignItems: 'center' },
  qualOpt: { padding: 14, borderRadius: 12, borderWidth: 2, marginBottom: 8 },
  storageRow: { flexDirection: 'row', justifyContent: 'space-between', paddingVertical: 8, borderBottomWidth: 1, borderBottomColor: '#F5F5F5' },
});
