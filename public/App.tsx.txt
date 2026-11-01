/**
 * @license
 * SPDX-License-Identifier: Apache-2.0
 */

import React, { useState, useMemo } from 'react';
import { 
  Phone, 
  MessageCircle, 
  Calculator, 
  ShieldCheck, 
  Clock, 
  Truck, 
  CheckCircle2, 
  Copy, 
  ChevronRight, 
  Calendar, 
  MapPin, 
  Package, 
  ArrowRight, 
  ZoomIn, 
  X, 
  Star, 
  HelpCircle, 
  Search, 
  Check, 
  Sparkles,
  Layers,
  FileText,
  Upload,
  Plus,
  Trash2,
  Camera,
  Filter,
  Image as ImageIcon
} from 'lucide-react';

// Real Images & Generated Assets
import heroTruckImg from './assets/images/hero_moving_truck_1793498561906.jpg';
import pickupBoxImg from './assets/images/truck_pickup_box_1793498562950.jpg';
import moverTeamImg from './assets/images/moving_service_team_1793498562993.jpg';
import sixWheelerImg from './assets/images/truck_six_wheeler_1793498563037.jpg';

// Gallery Item Interface
interface GalleryItem {
  id: string;
  url: string;
  title: string;
  desc: string;
  category: string;
  date?: string;
  isUserAdded?: boolean;
}

// Service Types Configuration
interface ServiceType {
  id: number;
  name: string;
  desc: string;
  basePrice: number;
  baseKm: number;
  perKm: number;
  icon: string;
  recommendedFor: string;
}

const SERVICE_TYPES: ServiceType[] = [
  {
    id: 0,
    name: 'ขนย้ายหอพัก / คอนโด',
    desc: 'เหมาะสำหรับเตียง 3.5-6 ฟุต ตู้เย็น เครื่องซักผ้า กล่องพัสดุ สัมภาระทั่วไป',
    basePrice: 500,
    baseKm: 10,
    perKm: 18,
    icon: '🏠',
    recommendedFor: 'นักศึกษา คนทำงาน ย้ายห้อง ย้ายคอนโด'
  },
  {
    id: 1,
    name: 'ขนย้ายบ้าน / ของชิ้นใหญ่',
    desc: 'เฟอร์นิเจอร์ โซฟา โต๊ะทานข้าว ชั้นวาง ทีวี สินค้าขนาดใหญ่พิเศษ',
    basePrice: 700,
    baseKm: 10,
    perKm: 20,
    icon: '🛋️',
    recommendedFor: 'ทาวน์โฮม บ้านเดี่ยว ย้ายออฟฟิศ'
  },
  {
    id: 2,
    name: 'ขนส่งสินค้าทั่วไป / ออกบูธ',
    desc: 'ส่งสินค้าโรงงาน พาเลท อุปกรณ์จัดอีเวนต์ วิ่งตรงปลายทางทันที ไม่รอรอบ',
    basePrice: 600,
    baseKm: 10,
    perKm: 18,
    icon: '📦',
    recommendedFor: 'ร้านค้า โรงงาน ธุรกิจ SME งานออกบูธ'
  },
  {
    id: 3,
    name: 'ขนย้ายมอเตอร์ไซค์ / บิ๊กไบค์',
    desc: 'ขึ้นสะพานมาตรฐาน ล็อกสายรัดสเตย์ 4 จุด แน่นหนา ไม่ล้ม ไม่เป็นรอย',
    basePrice: 550,
    baseKm: 10,
    perKm: 18,
    icon: '🏍️',
    recommendedFor: 'บิ๊กไบค์ มอเตอร์ไซค์คลาสสิก เวสป้า รถสกู๊ตเตอร์'
  }
];

export default function App() {
  // State for Calculator
  const [selectedServiceId, setSelectedServiceId] = useState<number>(0);
  const [distanceKm, setDistanceKm] = useState<number>(20);
  const [helperOption, setHelperOption] = useState<number>(0); // 0: no extra helper (driver assists), 1: 1 helper, 2: 2 helpers
  const [floorOption, setFloorOption] = useState<number>(0); // 0: elevator/1st floor, 1: 2nd floor, 2: 3rd floor, 3: 4th-5th floor
  const [includeWrapping, setIncludeWrapping] = useState<boolean>(true); // free plastic wrap & stay straps

  // Modals & UI States
  const [isBookingModalOpen, setIsBookingModalOpen] = useState<boolean>(false);
  const [bookingConfirmed, setBookingConfirmed] = useState<boolean>(false);
  const [bookingReference, setBookingReference] = useState<string>('');
  const [lightboxImg, setLightboxImg] = useState<{ url: string; title: string } | null>(null);
  const [toastMessage, setToastMessage] = useState<string | null>(null);

  // Tracking Simulation
  const [trackingInput, setTrackingInput] = useState<string>('JT-2026-8812');
  const [activeTrackingResult, setActiveTrackingResult] = useState<{
    code: string;
    driverName: string;
    plate: string;
    status: string;
    step: number;
    eta: string;
    route: string;
  } | null>({
    code: 'JT-2026-8812',
    driverName: 'คุณเจ (พนักงานขับรถมืออาชีพ)',
    plate: '3ฒข-8492 กรุงเทพมหานคร (กระบะตู้ทึบสูง 2.10 ม.)',
    status: 'กำลังเดินทางสู่จุดรับของ (ต้นทาง)',
    step: 2,
    eta: 'อีกประมาณ 20 นาทีถึงจุดนัดหมาย',
    route: 'บางนา-ตราด กม.14 ➔ รังสิต-ปทุมธานี'
  });

  // Booking Form State
  const [bookingForm, setBookingForm] = useState({
    customerName: '',
    phone: '',
    lineId: '',
    serviceDate: '',
    serviceTime: '',
    pickupLocation: '',
    dropoffLocation: '',
    notes: ''
  });

  // Toast Helper
  const showToast = (msg: string) => {
    setToastMessage(msg);
    setTimeout(() => {
      setToastMessage(null);
    }, 3200);
  };

  // Copy Phone Helper
  const copyToClipboard = (text: string, label: string) => {
    navigator.clipboard.writeText(text);
    showToast(`คัดลอก${label} (${text}) เรียบร้อยแล้ว!`);
  };

  // Pricing Calculation Logic
  const calculation = useMemo(() => {
    const service = SERVICE_TYPES.find(s => s.id === selectedServiceId) || SERVICE_TYPES[0];
    let base = service.basePrice;
    
    // Distance pricing
    let extraKm = Math.max(0, distanceKm - service.baseKm);
    let distanceCost = extraKm * service.perKm;

    // Helper cost (0: driver helps at tail, 1: +300, 2: +600)
    let helperCost = helperOption === 1 ? 300 : helperOption === 2 ? 600 : 0;

    // Floor cost (stairs without elevator)
    let floorCost = floorOption === 1 ? 100 : floorOption === 2 ? 200 : floorOption === 3 ? 300 : 0;

    let total = base + distanceCost + helperCost + floorCost;

    return {
      service,
      baseCost: base,
      distanceCost,
      helperCost,
      floorCost,
      total
    };
  }, [selectedServiceId, distanceKm, helperOption, floorOption]);

  // Handle Booking Submit
  const handleBookingSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!bookingForm.customerName || !bookingForm.phone || !bookingForm.pickupLocation) {
      alert('กรุณากรอกข้อมูลที่จำเป็น: ชื่อผู้ติดต่อ, เบอร์โทร และจุดรับของ');
      return;
    }
    const refCode = `JT-2026-${Math.floor(1000 + Math.random() * 9000)}`;
    setBookingReference(refCode);
    setBookingConfirmed(true);
    showToast(`บันทึกการจองสำเร็จ! รหัสอ้างอิง: ${refCode}`);
  };

  // Customer cargo photos for quote estimation
  const [cargoPhotos, setCargoPhotos] = useState<string[]>([]);

  // Format pre-filled LINE message
  const getLineShareUrl = () => {
    const text = `สวัสดีครับ สนใจจองรถรับจ้าง JT TRANSPORT 2026 TH
- บริการ: ${calculation.service.name}
- ระยะทางประมาณ: ${distanceKm} กม.
- ผู้ช่วยยก: ${helperOption === 0 ? 'คนขับช่วยจับของ (ฟรี)' : helperOption === 1 ? 'เพิ่มคนยก 1 คน (+300 บ.)' : 'เพิ่มคนยก 2 คน (+600 บ.)'}
- สถานที่: ${floorOption === 0 ? 'มีลิฟต์/ชั้น 1' : `บันไดชั้น ${floorOption + 1}`}
- ราคาประเมินเบื้องต้น: ${calculation.total.toLocaleString()} บาท${cargoPhotos.length > 0 ? `\n- มีรูปถ่ายสิ่งของที่จะย้ายแนบไว้ ${cargoPhotos.length} รูป (พร้อมส่งให้แอดมินดูในแชทนี้ครับ)` : ''}
รบกวนเช็คคิวงานและคอนเฟิร์มราคาด้วยครับ ขอบคุณครับ`;
    return `https://line.me/ti/p/~max_milin2564?text=${encodeURIComponent(text)}`;
  };

  // Gallery items initial data
  const INITIAL_GALLERY: GalleryItem[] = [
    {
      id: 'g-1',
      url: pickupBoxImg,
      title: 'รถกระบะตู้ทึบหลังคาสูง 2.10 ม. บรรจุของจุใจ',
      desc: 'ขนย้ายเตียง 6 ฟุต ตู้เย็น โซฟาในแนวตั้งได้สบาย ซีลมิดชิดกันฝน 100%',
      category: 'ย้ายหอ/คอนโด',
      date: 'ล่าสุด'
    },
    {
      id: 'g-2',
      url: moverTeamImg,
      title: 'ทีมงานช่วยยกของมืออาชีพ ซีนพลาสติกกันรอย',
      desc: 'ช่วยยกอย่างระมัดระวัง มีฟิล์มยืดแรปเฟอร์นิเจอร์ป้องกันรอยขูดขีดฟรี',
      category: 'ย้ายหอ/คอนโด',
      date: 'ล่าสุด'
    },
    {
      id: 'g-3',
      url: heroTruckImg,
      title: 'รถรับจ้างวิ่งตรงถึงหน้าบ้าน 77 จังหวัดทั่วไทย',
      desc: 'สแตนด์บายพร้อมวิ่ง 24 ชม. ส่งตรงถึงมือผู้รับปลายทาง ไม่ดึงเวลา',
      category: 'ย้ายบ้าน/ออฟฟิศ',
      date: 'ล่าสุด'
    },
    {
      id: 'g-4',
      url: sixWheelerImg,
      title: 'บริการขนส่งสินค้าโรงงาน & พาเลทออกบูธ',
      desc: 'รองรับการกระจายสินค้าด่วน คลังสินค้า ไบเทค อิมแพ็ค ทันเปิดงานแน่นอน',
      category: 'ขนส่งสินค้าโรงงาน',
      date: 'ล่าสุด'
    },
    {
      id: 'g-5',
      url: pickupBoxImg,
      title: 'ขนย้ายมอเตอร์ไซค์ & บิ๊กไบค์ ล็อกสเตย์ 4 จุด',
      desc: 'มีสะพานขึ้น-ลงมาตรฐาน ใช้สายรัดสเตย์แบบเฟืองล็อกแน่นหนา ไม่ล้มไม่เอียง',
      category: 'ย้ายมอเตอร์ไซค์บิ๊กไบค์',
      date: 'ล่าสุด'
    },
    {
      id: 'g-6',
      url: sixWheelerImg,
      title: 'งานขนย้ายบ้านเดี่ยว ทาวน์โฮม และออฟฟิศ',
      desc: 'ทีมงานครบชุดพร้อมรถเข็นของและอุปกรณ์ป้องกันความเสียหายทุกขั้นตอน',
      category: 'ย้ายบ้าน/ออฟฟิศ',
      date: 'ล่าสุด'
    },
    {
      id: 'g-7',
      url: moverTeamImg,
      title: 'อุปกรณ์เซฟตี้ประจำรถ: สายรัดสเตย์ & รถเข็น',
      desc: 'มีสายรัดล็อกเฟืองเกรดอุตสาหกรรม ผ้าห่มนวมกันกระแทกครบทุกคัน',
      category: 'รถและอุปกรณ์เซฟตี้',
      date: 'อุปกรณ์มาตรฐาน'
    },
    {
      id: 'g-8',
      url: heroTruckImg,
      title: 'ความสูงตู้ 2.10 เมตร มาตรฐานโรงงาน',
      desc: 'คนขับสามารถยืนจัดของในตู้ได้ จัดเรียงของได้เป็นระเบียบ จุได้มากกว่า 2 เท่า',
      category: 'รถและอุปกรณ์เซฟตี้',
      date: 'สเปกรถ'
    }
  ];

  // Gallery state with persistence
  const [galleryCategory, setGalleryCategory] = useState<string>('ทั้งหมด');
  const [galleryList, setGalleryList] = useState<GalleryItem[]>(() => {
    try {
      const saved = localStorage.getItem('jt_custom_photos');
      if (saved) {
        const parsed: GalleryItem[] = JSON.parse(saved);
        return [...parsed, ...INITIAL_GALLERY];
      }
    } catch {
      // fallback
    }
    return INITIAL_GALLERY;
  });

  // Modal to upload a new real work photo to gallery
  const [isAddPhotoModalOpen, setIsAddPhotoModalOpen] = useState<boolean>(false);
  const [newPhotoData, setNewPhotoData] = useState({
    title: '',
    category: 'ย้ายหอ/คอนโด',
    desc: '',
    imagePreview: ''
  });

  // Handle uploading photo to gallery
  const handleAddPhotoSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!newPhotoData.imagePreview || !newPhotoData.title) {
      alert('กรุณาเลือกไฟล์รูปภาพและกรอกชื่อผลงาน');
      return;
    }
    const newEntry: GalleryItem = {
      id: `custom-${Date.now()}`,
      url: newPhotoData.imagePreview,
      title: newPhotoData.title,
      desc: newPhotoData.desc || 'ผลงานการขนส่งจริงจาก JT TRANSPORT',
      category: newPhotoData.category,
      date: 'อัปเดตใหม่',
      isUserAdded: true
    };
    const updated = [newEntry, ...galleryList];
    setGalleryList(updated);
    
    // Save custom items to localStorage
    try {
      const customItems = updated.filter(i => i.isUserAdded);
      localStorage.setItem('jt_custom_photos', JSON.stringify(customItems));
    } catch {
      // ignore quota
    }

    setNewPhotoData({
      title: '',
      category: 'ย้ายหอ/คอนโด',
      desc: '',
      imagePreview: ''
    });
    setIsAddPhotoModalOpen(false);
    showToast('เพิ่มรูปภาพผลงานเข้าแกลเลอรีสำเร็จ!');
  };

  // Handle photo file selection for gallery
  const handlePhotoFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setNewPhotoData(prev => ({ ...prev, imagePreview: reader.result as string }));
      };
      reader.readAsDataURL(file);
    }
  };

  // Handle customer cargo photo upload
  const handleCargoPhotoChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const files = e.target.files;
    if (files && files.length > 0) {
      Array.from(files).forEach(file => {
        const reader = new FileReader();
        reader.onloadend = () => {
          if (reader.result) {
            setCargoPhotos(prev => [...prev, reader.result as string]);
          }
        };
        reader.readAsDataURL(file);
      });
      showToast(`แนบรูปสิ่งของแล้ว ${files.length} รูป`);
    }
  };

  // Remove cargo photo
  const handleRemoveCargoPhoto = (index: number) => {
    setCargoPhotos(prev => prev.filter((_, i) => i !== index));
  };

  // Delete user added photo from gallery
  const handleDeletePhoto = (id: string, e: React.MouseEvent) => {
    e.stopPropagation();
    if (confirm('ต้องการลบรูปภาพนี้ออกจากแกลเลอรีใช่หรือไม่?')) {
      const updated = galleryList.filter(item => item.id !== id);
      setGalleryList(updated);
      try {
        const customItems = updated.filter(i => i.isUserAdded);
        localStorage.setItem('jt_custom_photos', JSON.stringify(customItems));
      } catch {
        // ignore
      }
      showToast('ลบรูปภาพเรียบร้อยแล้ว');
    }
  };

  // Filtered gallery items
  const filteredGallery = useMemo(() => {
    if (galleryCategory === 'ทั้งหมด') return galleryList;
    return galleryList.filter(item => item.category === galleryCategory);
  }, [galleryList, galleryCategory]);

  // FAQ items
  const [activeFaq, setActiveFaq] = useState<number | null>(0);
  const faqs = [
    {
      q: 'ฝนตกหนัก ข้าวของด้านในจะเปียกไหม?',
      a: 'ไม่เปียกแน่นอน 100% ครับ ตู้บรรทุกของ JT TRANSPORT เป็นตู้ทึบอลูมิเนียมมาตรฐานโรงงาน พร้อมซีลยางกันน้ำอย่างหนารอบขอบประตู ปิดมิดชิดกันน้ำ ฝน ฝุ่น ได้อย่างสมบูรณ์แบบ'
    },
    {
      q: 'มีทีมงานช่วยยกของให้ด้วยหรือไม่?',
      a: 'พนักงานขับรถยินดีช่วยจัดและยกของท้ายรถฟรีครับ และหากลูกค้ามีของชิ้นใหญ่หรือต้องการผู้ช่วยยกเพิ่มเติม สามารถเลือกบริการเสริม "เด็กยกของ" ในราคาเริ่มต้นเพียง 300 บาท/คน'
    },
    {
      q: 'ต้องจองคิวล่วงหน้านานแค่ไหน?',
      a: 'สามารถจองล่วงหน้า 1-3 วันเพื่อให้ได้เวลาที่ต้องการแน่นอน หรือในกรณีงานด่วนฉุกเฉิน สามารถโทรสอบถามได้ตลอด 24 ชม. หากมีรถสแตนด์บายพร้อมออกวิ่งได้ภายใน 30-60 นาทีครับ'
    },
    {
      q: 'รับวิ่งงานต่างจังหวัดทั่วประเทศไหม?',
      a: 'รับวิ่งทั่วไทย 77 จังหวัดครับ ไม่ว่าจะเป็นภาคเหนือ ภาคอีสาน ภาคใต้ หรือภาคตะวันออก เราวิ่งตรง ส่งด่วน ไม่ดึงเวลา ข้าวของส่งถึงมือผู้รับปลายทางอย่างปลอดภัยแน่นอน'
    },
    {
      q: 'ขนย้ายมอเตอร์ไซค์ หรือบิ๊กไบค์ ปลอดภัยไหม?',
      a: 'ปลอดภัยมากครับ เรามีสะพานขึ้น-ลงมาตรฐาน และใช้สายรัดสเตย์แบบเฟืองล็อกแน่นหนา 4 จุด ยึดติดกับโครงตู้แน่นหนา รถไม่เอียง ไม่ล้ม และไม่เป็นรอยตลอดการเดินทาง'
    }
  ];

  return (
    <div className="min-h-screen bg-[#061633] text-slate-100 bg-grid-pattern relative pb-24 md:pb-0 font-kanit">
      
      {/* Toast Notification */}
      {toastMessage && (
        <div className="fixed top-20 right-4 md:right-8 z-50 bg-emerald-600 text-white px-5 py-3 rounded-xl shadow-2xl flex items-center gap-3 animate-fade-in border border-emerald-400">
          <CheckCircle2 className="w-5 h-5 shrink-0" />
          <span className="text-sm font-medium">{toastMessage}</span>
        </div>
      )}

      {/* Top Header */}
      <header className="sticky top-0 z-40 bg-[#061633]/90 backdrop-blur-md border-b border-[#1E3A74]">
        <div className="max-w-6xl mx-auto px-4 h-18 flex items-center justify-between">
          
          {/* Brand Wordmark */}
          <a href="#home" className="flex items-center gap-3 group">
            <div className="w-11 h-11 rounded-xl bg-gradient-to-br from-amber-400 via-[#2F80FF] to-blue-700 flex items-center justify-center font-black text-xl text-slate-950 shadow-lg shadow-blue-500/20 group-hover:scale-105 transition-transform">
              JT
            </div>
            <div>
              <div className="flex items-center gap-2">
                <span className="font-extrabold text-xl tracking-tight text-white">
                  JT <span className="text-[#2F80FF]">TRANSPORT</span>
                </span>
                <span className="text-[10px] font-bold px-2 py-0.5 rounded-full border border-blue-500/50 text-[#2F80FF] bg-blue-500/10">
                  2026
                </span>
              </div>
              <p className="text-[11px] text-slate-400 -mt-1 font-normal">
                รถรับจ้างทั่วไทยส่งสินค้า ขนของ ย้ายบ้าน 24 ชม.
              </p>
            </div>
          </a>

          {/* Desktop Navigation Links */}
          <nav className="hidden lg:flex items-center gap-7 text-sm font-medium text-slate-300">
            <a href="#services" className="hover:text-blue-400 transition-colors">บริการของเรา</a>
            <a href="#calculator" className="hover:text-blue-400 transition-colors">ประเมินราคา</a>
            <a href="#specs" className="hover:text-blue-400 transition-colors">สเปกรถตู้ทึบ</a>
            <a href="#gallery" className="hover:text-blue-400 transition-colors">ผลงานจริง</a>
            <a href="#tracking" className="hover:text-blue-400 transition-colors">เช็คสถานะงาน</a>
            <a href="#faq" className="hover:text-blue-400 transition-colors">คำถามที่พบบ่อย</a>
          </nav>

          {/* Header Action Button & Hotlines */}
          <div className="hidden sm:flex items-center gap-4">
            <div className="text-right">
              <a href="tel:+66879209677" className="text-sm font-bold text-[#2F80FF] hover:underline flex items-center gap-1.5 justify-end">
                <Phone className="w-3.5 h-3.5" /> 087-920-9677
              </a>
              <span className="text-[11px] text-slate-400 block">
                สำรอง: <a href="tel:+66967744051" className="hover:text-slate-200">096-774-4051</a>
              </span>
            </div>
            <a 
              href="https://line.me/ti/p/~max_milin2564" 
              target="_blank" 
              rel="noopener noreferrer"
              className="inline-flex items-center gap-2 bg-[#00C853] hover:bg-[#00E676] text-slate-950 font-bold px-4 py-2.5 rounded-xl shadow-lg shadow-emerald-500/20 text-xs transition-transform active:scale-95"
            >
              <MessageCircle className="w-4 h-4" />
              แชทไลน์จองคิว
            </a>
          </div>
        </div>
      </header>

      {/* Hero Section */}
      <section id="home" className="relative pt-10 pb-16 md:py-20 overflow-hidden">
        {/* Glow ambient background */}
        <div className="absolute top-1/4 left-1/2 -translate-x-1/2 w-[600px] h-[340px] bg-blue-600/15 blur-[120px] rounded-full pointer-events-none" />
        
        <div className="max-w-4xl mx-auto px-4 text-center relative z-10">
          
          {/* Availability Pill */}
          <div className="inline-flex items-center gap-2.5 px-4 py-2 rounded-full bg-[#0C2250]/90 border border-[#2B4C94] text-xs font-medium text-slate-200 mb-6 shadow-md">
            <span className="w-2.5 h-2.5 rounded-full bg-[#00C853] shadow-[0_0_10px_#00C853] animate-pulse" />
            <span>พร้อมวิ่งงานรับส่งสินค้า ย้ายหอ ย้ายบ้าน ทั่วไทย 24 ชม.</span>
          </div>

          {/* Main Hero Headline */}
          <h1 className="text-3xl sm:text-5xl md:text-6xl font-black tracking-tight text-white leading-tight">
            JT TRANSPORT 2026 TH
          </h1>
          <h2 className="text-xl sm:text-3xl md:text-4xl font-extrabold mt-2 bg-gradient-to-r from-[#2F80FF] via-blue-300 to-[#FFC400] bg-clip-text text-transparent">
            รถรับจ้างทั่วไทยส่งสินค้า
          </h2>

          <p className="max-w-2xl mx-auto mt-4 text-sm sm:text-base text-slate-300 font-light leading-relaxed">
            บริการรถรับจ้างทั่วไทยส่งสินค้า ขนย้ายหอพัก คอนโด บ้านเดี่ยว ขนย้ายเฟอร์นิเจอร์ ซีนพลาสติกกันรอย และย้ายมอเตอร์ไซค์บิ๊กไบค์{' '}
            <strong className="text-white font-medium">รถกระบะ ตู้ทึบหลังคาสูง 2.10 เมตร</strong> กันฝน 100% สายรัดสเตย์ยึดแน่นหนา ปลอดภัย วิ่งตรง ไม่ดึงเวลา
          </p>

          {/* High-Intent Contact Callout Card */}
          <div className="max-w-xl mx-auto mt-8 bg-[#0C2250]/95 border border-[#1E3A74] rounded-2xl p-5 sm:p-6 shadow-2xl backdrop-blur-md">
            <p className="text-amber-400 font-semibold text-xs sm:text-sm mb-4 flex items-center justify-center gap-2">
              <Sparkles className="w-4 h-4" /> ประเมินราคาไว ติดต่อแอดมินตอบกลับภายใน 3 นาที
            </p>

            {/* Direct Phone Numbers Grid */}
            <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">
              
              {/* Phone 1 */}
              <a 
                href="tel:+66879209677"
                className="flex items-center justify-center gap-3 p-3.5 rounded-xl bg-gradient-to-r from-[#2F80FF] to-blue-700 text-white font-bold shadow-lg shadow-blue-500/25 hover:brightness-110 transition-all text-left group"
              >
                <div className="w-10 h-10 rounded-lg bg-white/20 flex items-center justify-center text-xl shrink-0 group-hover:scale-110 transition-transform">
                  📞
                </div>
                <div>
                  <span className="block text-[11px] font-normal text-blue-100">เบอร์โทรหลัก (เบอร์แรก)</span>
                  <span className="text-lg tracking-wider font-extrabold">087-920-9677</span>
                </div>
              </a>

              {/* Phone 2 */}
              <a 
                href="tel:+66967744051"
                className="flex items-center justify-center gap-3 p-3.5 rounded-xl bg-[#16346F] border border-[#2B4C94] text-white font-bold hover:border-blue-400 transition-all text-left group"
              >
                <div className="w-10 h-10 rounded-lg bg-blue-500/20 flex items-center justify-center text-xl shrink-0 group-hover:scale-110 transition-transform">
                  ☎️
                </div>
                <div>
                  <span className="block text-[11px] font-normal text-slate-400">เบอร์โทรสำรอง</span>
                  <span className="text-lg tracking-wider font-extrabold">096-774-4051</span>
                </div>
              </a>
            </div>

            {/* Quick Copy Buttons */}
            <div className="flex gap-2.5 mt-3">
              <button 
                type="button" 
                onClick={() => copyToClipboard('0879209677', 'เบอร์โทรหลัก')}
                className="flex-1 py-2 px-3 rounded-lg border border-[#2B4C94] bg-[#16346F]/80 hover:bg-[#1E3A74] text-xs text-slate-200 font-medium flex items-center justify-center gap-1.5 transition-colors cursor-pointer"
              >
                <Copy className="w-3.5 h-3.5 text-blue-400" /> คัดลอกเบอร์หลัก
              </button>
              <button 
                type="button" 
                onClick={() => copyToClipboard('0967744051', 'เบอร์โทรสำรอง')}
                className="flex-1 py-2 px-3 rounded-lg border border-[#2B4C94] bg-[#16346F]/80 hover:bg-[#1E3A74] text-xs text-slate-200 font-medium flex items-center justify-center gap-1.5 transition-colors cursor-pointer"
              >
                <Copy className="w-3.5 h-3.5 text-blue-400" /> คัดลอกเบอร์สำรอง
              </button>
            </div>

            {/* Line ID Link */}
            <a 
              href="https://line.me/ti/p/~max_milin2564" 
              target="_blank" 
              rel="noopener noreferrer"
              className="mt-3 block p-2.5 rounded-xl bg-emerald-500/10 border border-dashed border-emerald-500/50 text-emerald-300 text-sm font-semibold hover:bg-emerald-500/20 transition-colors"
            >
              💬 LINE ID: <strong className="text-white font-extrabold tracking-wide">max_milin2564</strong> (คลิกเพื่อแอดไลน์)
            </a>
          </div>

          {/* Action CTAs */}
          <div className="flex flex-col sm:flex-row items-center justify-center gap-3.5 mt-6">
            <a 
              href="https://line.me/ti/p/~max_milin2564" 
              target="_blank" 
              rel="noopener noreferrer"
              className="w-full sm:w-auto inline-flex items-center justify-center gap-2 bg-[#00C853] hover:bg-[#00E676] text-slate-950 font-extrabold px-6 py-3.5 rounded-xl shadow-xl shadow-emerald-500/25 transition-transform active:scale-95 text-sm"
            >
              <MessageCircle className="w-5 h-5" />
              แชทไลน์จองคิว / สอบถามราคา
            </a>
            <a 
              href="#calculator"
              className="w-full sm:w-auto inline-flex items-center justify-center gap-2 bg-[#0C2250] border border-[#1E3A74] hover:border-blue-400 text-slate-200 font-bold px-6 py-3.5 rounded-xl transition-colors text-sm"
            >
              <Calculator className="w-5 h-5 text-blue-400" />
              ประเมินราคาขนย้ายเบื้องต้น
            </a>
            <button
              onClick={() => setIsBookingModalOpen(true)}
              className="w-full sm:w-auto inline-flex items-center justify-center gap-2 bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-500 hover:to-indigo-500 text-white font-bold px-6 py-3.5 rounded-xl shadow-lg shadow-blue-600/30 transition-transform active:scale-95 text-sm cursor-pointer"
            >
              <Calendar className="w-5 h-5 text-amber-300" />
              จองคิวออนไลน์ด่วน
            </button>
          </div>

          {/* Clean Graphic Hero Asset */}
          <div className="mt-10 max-w-lg mx-auto rounded-3xl overflow-hidden border border-[#2B4C94] shadow-2xl shadow-blue-500/20 ring-1 ring-blue-500/30 relative group">
            <img 
              src={heroTruckImg} 
              alt="รถรับจ้างทั่วไทยส่งสินค้า JT TRANSPORT 2026" 
              className="w-full h-auto object-cover group-hover:scale-105 transition-transform duration-700"
            />
            <div className="absolute inset-0 bg-gradient-to-t from-[#061633] via-[#061633]/20 to-transparent opacity-90 pointer-events-none" />
            <div className="absolute bottom-3.5 left-4 right-4 flex items-center justify-between text-xs text-slate-200 font-medium">
              <span className="flex items-center gap-1.5">
                <Truck className="w-4 h-4 text-blue-400" />
                รถรับจ้างทั่วไทยส่งสินค้า พร้อมทีมงานมืออาชีพ
              </span>
              <span className="text-amber-400 font-bold bg-amber-400/10 px-2 py-0.5 rounded-full border border-amber-400/30">
                วิ่งตรง 77 จังหวัด
              </span>
            </div>
          </div>

          {/* Quick Stats Strip */}
          <div className="grid grid-cols-3 gap-3 max-w-xl mx-auto mt-8">
            <div className="bg-[#0C2250]/70 border border-[#1E3A74] rounded-xl p-3 text-center">
              <b className="block text-[#2F80FF] text-xl font-black">24 ชม.</b>
              <span className="text-[11px] text-slate-400">พร้อมให้บริการทุกวัน</span>
            </div>
            <div className="bg-[#0C2250]/70 border border-[#1E3A74] rounded-xl p-3 text-center">
              <b className="block text-emerald-400 text-xl font-black">100%</b>
              <span className="text-[11px] text-slate-400">กันน้ำ กันฝน กันฝุ่น</span>
            </div>
            <div className="bg-[#0C2250]/70 border border-[#1E3A74] rounded-xl p-3 text-center">
              <b className="block text-amber-400 text-xl font-black">3 นาที</b>
              <span className="text-[11px] text-slate-400">ตอบกลับประเมินไว</span>
            </div>
          </div>

        </div>
      </section>

      {/* Services Section */}
      <section id="services" className="py-14 bg-[#030E28]/60 border-y border-[#1E3A74]">
        <div className="max-w-6xl mx-auto px-4">
          <div className="text-center mb-10">
            <span className="text-[#2F80FF] font-bold text-xs tracking-widest uppercase block mb-1">
              JT Transport Services
            </span>
            <h3 className="text-2xl sm:text-3xl font-black text-white">
              บริการรถกระบะตู้ทึบครอบคลุมทุกงานขนย้าย
            </h3>
          </div>

          <div className="grid grid-cols-1 md:grid-cols-3 gap-5">
            
            {/* Card 1 */}
            <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-6 hover:border-blue-400/60 transition-all hover:-translate-y-1 shadow-xl group">
              <div className="w-14 h-14 rounded-2xl bg-blue-500/10 flex items-center justify-center text-3xl mb-4 group-hover:scale-110 transition-transform">
                🏠
              </div>
              <h4 className="text-lg font-bold text-white mb-2">ขนย้ายหอพัก / คอนโด / บ้าน</h4>
              <p className="text-sm text-slate-300 font-light leading-relaxed mb-4">
                ทีมงานช่วยยกของอย่างทะนุถนอม พร้อมซีนพลาสติกกันรอยเฟอร์นิเจอร์ เตียง โซฟา ตู้เย็น ปลอดภัย สินค้าไม่กระแทกเสียหาย
              </p>
              <ul className="text-xs text-slate-400 space-y-1.5 border-t border-[#1E3A74]/80 pt-3">
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> เตียง 3.5 - 6 ฟุต พร้อมที่นอน</li>
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> ตู้เย็น เครื่องซักผ้า ทีวี โซฟา</li>
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> กล่องพัสดุและกระเป๋าสัมภาระ</li>
              </ul>
            </div>

            {/* Card 2 */}
            <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-6 hover:border-blue-400/60 transition-all hover:-translate-y-1 shadow-xl group">
              <div className="w-14 h-14 rounded-2xl bg-blue-500/10 flex items-center justify-center text-3xl mb-4 group-hover:scale-110 transition-transform">
                📦
              </div>
              <h4 className="text-lg font-bold text-white mb-2">ขนส่งสินค้าโรงงาน / ออกบูธ</h4>
              <p className="text-sm text-slate-300 font-light leading-relaxed mb-4">
                บริการรับส่งสินค้าด่วน ตู้ทึบหลังคาสูง กันน้ำกันฝน 100% วิ่งตรงส่งมอบถึงปลายทางทันที ไม่รอรอบ ไม่ดึงเวลา
              </p>
              <ul className="text-xs text-slate-400 space-y-1.5 border-t border-[#1E3A74]/80 pt-3">
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> สินค้าโรงงาน พาเลท อะไหล่</li>
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> อุปกรณ์จัดบูธ ป้ายไฟ โครงสร้าง</li>
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> กระจายสินค้าด่วน กทม.และตจว.</li>
              </ul>
            </div>

            {/* Card 3 */}
            <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-6 hover:border-blue-400/60 transition-all hover:-translate-y-1 shadow-xl group">
              <div className="w-14 h-14 rounded-2xl bg-blue-500/10 flex items-center justify-center text-3xl mb-4 group-hover:scale-110 transition-transform">
                🏍️
              </div>
              <h4 className="text-lg font-bold text-white mb-2">ขนย้ายมอเตอร์ไซค์ / บิ๊กไบค์</h4>
              <p className="text-sm text-slate-300 font-light leading-relaxed mb-4">
                มีสะพานขึ้นลงมาตรฐาน ล็อกสายรัดสเตย์ 4 ทิศทาง แน่นหนา ไม่ล้ม ไม่ขูดขีด ปลอดภัยตลอดทุกโค้งทุกเส้นทาง
              </p>
              <ul className="text-xs text-slate-400 space-y-1.5 border-t border-[#1E3A74]/80 pt-3">
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> มอเตอร์ไซค์ทั่วไป & สกู๊ตเตอร์</li>
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> บิ๊กไบค์ 300cc - 1000cc+</li>
                <li className="flex items-center gap-2"><Check className="w-3.5 h-3.5 text-emerald-400" /> รถคลาสสิก เวสป้า ช็อปเปอร์</li>
              </ul>
            </div>

          </div>
        </div>
      </section>

      {/* Interactive Pricing Calculator Section */}
      <section id="calculator" className="py-16">
        <div className="max-w-3xl mx-auto px-4">
          
          <div className="text-center mb-8">
            <span className="text-[#2F80FF] font-bold text-xs tracking-widest uppercase block mb-1">
              Fast Estimate
            </span>
            <h3 className="text-2xl sm:text-3xl font-black text-white">
              ประเมินราคาค่าขนส่งเบื้องต้น
            </h3>
            <p className="text-xs sm:text-sm text-slate-400 mt-1">
              คำนวณราคาโปร่งใสตามระยะทางจริงและบริการเสริม ไม่มีค่าใช้จ่ายแอบแฝง
            </p>
          </div>

          <div className="bg-[#0C2250] border border-[#1E3A74] rounded-3xl p-6 sm:p-8 shadow-2xl relative overflow-hidden">
            <div className="absolute right-0 bottom-0 w-60 h-60 bg-blue-500/10 blur-3xl pointer-events-none rounded-full" />

            <div className="space-y-6 relative z-10">
              
              {/* Step 1: Service Type */}
              <div>
                <label className="block text-sm font-semibold text-slate-200 mb-2.5">
                  1. เลือกประเภทงานขนย้าย
                </label>
                <div className="grid grid-cols-1 sm:grid-cols-2 gap-2.5">
                  {SERVICE_TYPES.map(s => (
                    <button
                      key={s.id}
                      type="button"
                      onClick={() => setSelectedServiceId(s.id)}
                      className={`p-3.5 rounded-xl border text-left transition-all cursor-pointer flex items-start gap-3 ${
                        selectedServiceId === s.id
                          ? 'bg-blue-600/20 border-[#2F80FF] text-white shadow-md'
                          : 'bg-[#16346F]/60 border-[#2B4C94] text-slate-300 hover:border-slate-400'
                      }`}
                    >
                      <span className="text-2xl">{s.icon}</span>
                      <div className="min-w-0">
                        <span className="font-bold text-sm block">{s.name}</span>
                        <span className="text-[11px] text-slate-400 block line-clamp-1">{s.desc}</span>
                      </div>
                    </button>
                  ))}
                </div>
              </div>

              {/* Step 2: Distance Range */}
              <div>
                <div className="flex justify-between items-center mb-2">
                  <label className="text-sm font-semibold text-slate-200">
                    2. ระยะทางประมาณ (กิโลเมตร)
                  </label>
                  <span className="text-lg font-black text-[#2F80FF]">
                    {distanceKm} กม.
                  </span>
                </div>
                
                <input 
                  type="range" 
                  min="1" 
                  max="350" 
                  value={distanceKm}
                  onChange={(e) => setDistanceKm(Number(e.target.value))}
                  className="w-full accent-[#2F80FF] h-2 bg-[#16346F] rounded-lg cursor-pointer"
                />

                {/* Distance Quick Presets */}
                <div className="flex flex-wrap gap-2 mt-2.5">
                  <span className="text-[11px] text-slate-400 self-center">ตัวอย่างเส้นทาง:</span>
                  {[
                    { label: 'ใน กทม. (15 กม.)', km: 15 },
                    { label: 'กทม. - ปทุม/นนท์ (35 กม.)', km: 35 },
                    { label: 'กทม. - ชลบุรี (85 กม.)', km: 85 },
                    { label: 'กทม. - โคราช (220 กม.)', km: 220 },
                  ].map((preset, idx) => (
                    <button
                      key={idx}
                      type="button"
                      onClick={() => setDistanceKm(preset.km)}
                      className="text-[11px] px-2.5 py-1 rounded-md bg-[#16346F] hover:bg-[#1E3A74] text-blue-200 border border-[#2B4C94] transition-colors cursor-pointer"
                    >
                      {preset.label}
                    </button>
                  ))}
                </div>
              </div>

              {/* Step 3: Helpers & Floor */}
              <div className="grid grid-cols-1 sm:grid-cols-2 gap-4 pt-1">
                
                {/* Helper Options */}
                <div>
                  <label className="block text-xs font-semibold text-slate-300 mb-2">
                    3. ผู้ช่วยยกของ (ทีมงาน)
                  </label>
                  <select 
                    value={helperOption}
                    onChange={(e) => setHelperOption(Number(e.target.value))}
                    className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3.5 py-2.5 text-sm text-white focus:outline-none focus:border-[#2F80FF] cursor-pointer"
                  >
                    <option value={0}>คนขับช่วยจับของท้ายรถ (ฟรี ไม่บวกเพิ่ม)</option>
                    <option value={1}>คนขับ + ผู้ช่วยยก 1 คน (+300 บ.)</option>
                    <option value={2}>คนขับ + ผู้ช่วยยก 2 คน (+600 บ.)</option>
                  </select>
                </div>

                {/* Floor Options */}
                <div>
                  <label className="block text-xs font-semibold text-slate-300 mb-2">
                    4. ชั้น / บันได
                  </label>
                  <select 
                    value={floorOption}
                    onChange={(e) => setFloorOption(Number(e.target.value))}
                    className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3.5 py-2.5 text-sm text-white focus:outline-none focus:border-[#2F80FF] cursor-pointer"
                  >
                    <option value={0}>มีลิฟต์ขนของ / ชั้น 1 (ไม่มีบวกเพิ่ม)</option>
                    <option value={1}>ไม่มีลิฟต์ - บันไดชั้น 2 (+100 บ.)</option>
                    <option value={2}>ไม่มีลิฟต์ - บันไดชั้น 3 (+200 บ.)</option>
                    <option value={3}>ไม่มีลิฟต์ - บันไดชั้น 4-5 (+300 บ.)</option>
                  </select>
                </div>
              </div>

              {/* Free Protection Guarantee Badge */}
              <div className="p-3 rounded-xl bg-emerald-500/10 border border-emerald-500/30 flex items-center gap-2.5 text-xs text-emerald-300">
                <CheckCircle2 className="w-4 h-4 text-emerald-400 shrink-0" />
                <span>
                  <strong>ฟรี!</strong> บริการฟิล์มยืดซีนพลาสติกกันรอย + สายรัดสเตย์ล็อกแน่นหนา 4 จุดทุกเที่ยววิ่ง
                </span>
              </div>

              {/* Step 5: Cargo Photo Upload for Accurate Quote */}
              <div className="bg-[#16346F]/40 border border-[#2B4C94] rounded-2xl p-4">
                <div className="flex justify-between items-center mb-2">
                  <label className="text-xs font-semibold text-slate-200 flex items-center gap-1.5">
                    <Camera className="w-4 h-4 text-blue-400" />
                    5. แนบรูปถ่ายสิ่งของที่ต้องการย้าย (เลือกได้หลายรูปเพื่อประเมินราคาแม่นยำ)
                  </label>
                  {cargoPhotos.length > 0 && (
                    <span className="text-[11px] text-emerald-400 font-bold">
                      เลือกแล้ว {cargoPhotos.length} รูป
                    </span>
                  )}
                </div>

                <label className="flex items-center justify-center gap-2.5 border-2 border-dashed border-[#2B4C94] hover:border-[#2F80FF] bg-[#0C2250]/80 rounded-xl p-3 text-center cursor-pointer transition-colors group">
                  <input 
                    type="file" 
                    multiple 
                    accept="image/*" 
                    onChange={handleCargoPhotoChange}
                    className="hidden" 
                  />
                  <Upload className="w-4 h-4 text-blue-400 group-hover:scale-110 transition-transform" />
                  <span className="text-xs text-slate-300 group-hover:text-white">
                    แตะที่นี่เพื่อถ่ายรูปหรือเลือกรูปสิ่งของจากเครื่อง (เตียง, โซฟา, ตู้เย็น, กล่อง)
                  </span>
                </label>

                {/* Previews of attached cargo photos */}
                {cargoPhotos.length > 0 && (
                  <div className="flex flex-wrap gap-2.5 mt-3 pt-3 border-t border-[#1E3A74]">
                    {cargoPhotos.map((photo, pIdx) => (
                      <div key={pIdx} className="relative w-16 h-16 rounded-xl overflow-hidden border border-[#2B4C94] group">
                        <img src={photo} alt={`รูปสิ่งของที่ ${pIdx + 1}`} className="w-full h-full object-cover" />
                        <button
                          type="button"
                          onClick={() => handleRemoveCargoPhoto(pIdx)}
                          className="absolute top-0.5 right-0.5 bg-red-600/90 hover:bg-red-600 text-white rounded-full p-1 transition-transform cursor-pointer"
                          title="ลบรูปนี้"
                        >
                          <X className="w-3 h-3" />
                        </button>
                      </div>
                    ))}
                  </div>
                )}
              </div>

              {/* Result Calculation Box */}
              <div className="bg-[#04102A] border border-[#1E3A74] rounded-2xl p-5 flex flex-col sm:flex-row items-center justify-between gap-4">
                <div className="text-center sm:text-left">
                  <span className="text-xs text-slate-400 block">ราคาประเมินเบื้องต้น</span>
                  <div className="flex items-baseline gap-2 justify-center sm:justify-start">
                    <span className="text-3xl sm:text-4xl font-black text-[#2F80FF]">
                      {calculation.total.toLocaleString()}
                    </span>
                    <span className="text-sm font-bold text-slate-300">บาท</span>
                  </div>
                  <span className="text-[11px] text-slate-500 block mt-0.5">
                    (ค่ารถเริ่มต้น {calculation.baseCost} บ. + ค่าระยะทาง {calculation.distanceCost} บ. {calculation.helperCost > 0 ? `+ คนยก ${calculation.helperCost} บ.` : ''})
                  </span>
                </div>

                <div className="flex flex-col sm:flex-row gap-2 w-full sm:w-auto">
                  <a 
                    href={getLineShareUrl()} 
                    target="_blank" 
                    rel="noopener noreferrer"
                    className="inline-flex items-center justify-center gap-2 bg-[#00C853] hover:bg-[#00E676] text-slate-950 font-bold px-5 py-3 rounded-xl shadow-lg shadow-emerald-500/20 text-xs transition-transform active:scale-95 text-center"
                  >
                    <MessageCircle className="w-4 h-4" />
                    ส่งสเปกนี้ไปแชท LINE
                  </a>
                  <button
                    type="button"
                    onClick={() => setIsBookingModalOpen(true)}
                    className="inline-flex items-center justify-center gap-2 bg-[#16346F] hover:bg-[#1E3A74] border border-[#2B4C94] text-white font-bold px-4 py-3 rounded-xl text-xs transition-colors cursor-pointer"
                  >
                    จองคิวออนไลน์
                  </button>
                </div>
              </div>

              <p className="text-[11px] text-slate-400 text-center">
                * ราคาที่แสดงเป็นราคาประเมินเบื้องต้นตามระยะทางมาตรฐาน ราคาจริงอาจปรับลดหรือเพิ่มตามปริมาณสิ่งของจริงและเงื่อนไขหน้างาน
              </p>

            </div>
          </div>
        </div>
      </section>

      {/* Truck Specs & Why Choose JT */}
      <section id="specs" className="py-14 bg-[#030E28]/60 border-y border-[#1E3A74]">
        <div className="max-w-6xl mx-auto px-4">
          <div className="text-center mb-10">
            <span className="text-[#2F80FF] font-bold text-xs tracking-widest uppercase block mb-1">
              Why JT Transport
            </span>
            <h3 className="text-2xl sm:text-3xl font-black text-white">
              สเปกรถกระบะตู้ทึบ & มาตรฐานความปลอดภัย
            </h3>
          </div>

          <div className="grid grid-cols-2 sm:grid-cols-4 gap-3.5 mb-10">
            <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-4 text-center">
              <span className="text-2xl mb-1 block">📏</span>
              <b className="block text-white text-sm font-bold">ตู้ทึบสูง 2.10 ม.</b>
              <span className="text-xs text-slate-400">คนยืนจัดของในตู้ได้สบาย</span>
            </div>
            <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-4 text-center">
              <span className="text-2xl mb-1 block">🌧️</span>
              <b className="block text-white text-sm font-bold">กันฝน 100%</b>
              <span className="text-xs text-slate-400">ปิดมิดชิด ซีลยางแน่นหนา</span>
            </div>
            <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-4 text-center">
              <span className="text-2xl mb-1 block">🔒</span>
              <b className="block text-white text-sm font-bold">สายรัดสเตย์ล็อก</b>
              <span className="text-xs text-slate-400">ป้องกันของตกหล่นหรือล้ม</span>
            </div>
            <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-4 text-center">
              <span className="text-2xl mb-1 block">📦</span>
              <b className="block text-white text-sm font-bold">ซีนพลาสติกกันรอย</b>
              <span className="text-xs text-slate-400">แรปกันกระแทกให้ฟรี</span>
            </div>
          </div>

          {/* Detailed Dimensions Breakdown */}
          <div className="bg-[#0C2250] border border-[#1E3A74] rounded-3xl p-6 sm:p-8 flex flex-col md:flex-row items-center gap-8">
            <div className="w-full md:w-1/2">
              <span className="text-xs text-blue-400 font-bold uppercase tracking-wider block mb-1">
                Vehicle Dimensions & Capacity
              </span>
              <h4 className="text-2xl font-black text-white mb-4">
                ขนาดตู้บรรทุกมาตรฐาน รองรับน้ำหนัก 1.5 - 2.0 ตัน
              </h4>
              <p className="text-sm text-slate-300 font-light leading-relaxed mb-6">
                รถกระบะของ JT TRANSPORT ดัดแปลงตู้ทึบอลูมิเนียมเกรดพรีเมียม ความสูงพิเศษ 2.10 เมตร ช่วยให้สามารถขนเตียง 6 ฟุต ที่นอนหนา ตู้เสื้อผ้าขนาดใหญ่ และตู้เย็น 2 ประตูในแนวตั้งได้โดยไม่ต้องเอียง
              </p>

              <div className="space-y-3">
                <div className="flex justify-between items-center p-3 rounded-xl bg-[#16346F]/60 border border-[#2B4C94]">
                  <span className="text-sm text-slate-300">ความยาวกระบะตู้</span>
                  <span className="text-sm font-bold text-white">2.20 เมตร</span>
                </div>
                <div className="flex justify-between items-center p-3 rounded-xl bg-[#16346F]/60 border border-[#2B4C94]">
                  <span className="text-sm text-slate-300">ความกว้างกระบะตู้</span>
                  <span className="text-sm font-bold text-white">1.70 เมตร</span>
                </div>
                <div className="flex justify-between items-center p-3 rounded-xl bg-[#16346F]/60 border border-[#2B4C94]">
                  <span className="text-sm text-slate-300">ความสูงจากพื้นตู้ถึงเพดาน</span>
                  <span className="text-sm font-bold text-amber-400">2.10 เมตร (คนยืนได้)</span>
                </div>
              </div>
            </div>

            <div className="w-full md:w-1/2 rounded-3xl overflow-hidden border border-[#2B4C94] shadow-2xl shadow-blue-900/40 ring-1 ring-blue-500/20 group relative">
              <img 
                src={pickupBoxImg} 
                alt="ขนาดตู้ทึบกระบะ JT TRANSPORT รถรับจ้างทั่วไทยส่งสินค้า" 
                className="w-full h-auto object-cover group-hover:scale-105 transition-transform duration-700"
              />
              <div className="absolute inset-0 bg-gradient-to-t from-[#061633] via-transparent to-transparent opacity-60 pointer-events-none" />
              <div className="absolute bottom-3.5 left-4 right-4 text-xs font-medium text-slate-200 flex items-center justify-between">
                <span>ตู้ทึบมาตรฐานโรงงาน บรรจุของได้จุใจ</span>
                <span className="text-emerald-400 font-bold bg-emerald-500/10 px-2 py-0.5 rounded-full border border-emerald-500/30">
                  กันฝน 100%
                </span>
              </div>
            </div>
          </div>

        </div>
      </section>

      {/* Real Work Gallery with Lightbox & Category Filters */}
      <section id="gallery" className="py-16">
        <div className="max-w-6xl mx-auto px-4">
          <div className="flex flex-col md:flex-row md:items-end justify-between gap-4 mb-8">
            <div>
              <span className="text-[#2F80FF] font-bold text-xs tracking-widest uppercase block mb-1">
                Real Work Gallery & Fleet
              </span>
              <h3 className="text-2xl sm:text-3xl font-black text-white">
                ภาพผลงานการขนส่งจริงจากหน้างาน
              </h3>
              <p className="text-xs sm:text-sm text-slate-400 mt-1">
                รวมภาพงานย้ายหอ คอนโด บ้าน มอเตอร์ไซค์ และการจัดวางของในตู้ทึบจริง
              </p>
            </div>

            {/* Button to Upload New Photo */}
            <button
              type="button"
              onClick={() => setIsAddPhotoModalOpen(true)}
              className="inline-flex items-center gap-2 bg-[#16346F] hover:bg-[#1E3A74] border border-[#2B4C94] text-white font-bold px-4 py-2.5 rounded-xl text-xs transition-all shadow-md active:scale-95 cursor-pointer w-fit"
            >
              <Plus className="w-4 h-4 text-emerald-400" />
              <span>เพิ่มรูปภาพผลงานใหม่</span>
            </button>
          </div>

          {/* Category Filter Tabs */}
          <div className="flex flex-wrap items-center gap-2 mb-8">
            <span className="text-xs text-slate-400 flex items-center gap-1.5 mr-1">
              <Filter className="w-3.5 h-3.5 text-blue-400" /> หมวดหมู่:
            </span>
            {[
              'ทั้งหมด',
              'ย้ายหอ/คอนโด',
              'ย้ายบ้าน/ออฟฟิศ',
              'ขนส่งสินค้าโรงงาน',
              'ย้ายมอเตอร์ไซค์บิ๊กไบค์',
              'รถและอุปกรณ์เซฟตี้'
            ].map(cat => (
              <button
                key={cat}
                type="button"
                onClick={() => setGalleryCategory(cat)}
                className={`px-3.5 py-1.5 rounded-lg text-xs font-semibold transition-all cursor-pointer ${
                  galleryCategory === cat
                    ? 'bg-[#2F80FF] text-white shadow-md shadow-blue-500/25'
                    : 'bg-[#0C2250] hover:bg-[#16346F] text-slate-300 border border-[#1E3A74]'
                }`}
              >
                {cat}
              </button>
            ))}
          </div>

          {/* Gallery Grid */}
          <div className="grid grid-cols-2 sm:grid-cols-4 gap-4">
            {filteredGallery.map((item) => (
              <figure 
                key={item.id}
                onClick={() => setLightboxImg({ url: item.url, title: item.title })}
                className="group relative rounded-2xl overflow-hidden bg-[#0C2250] border border-[#1E3A74] hover:border-[#2F80FF] transition-all cursor-zoom-in hover:-translate-y-1 shadow-lg flex flex-col justify-between"
              >
                <div className="aspect-[4/3] overflow-hidden relative">
                  <img 
                    src={item.url} 
                    alt={item.title} 
                    className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
                  />
                  <div className="absolute inset-0 bg-blue-900/30 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center">
                    <ZoomIn className="w-8 h-8 text-white drop-shadow" />
                  </div>
                  
                  {/* Category Pill on Image */}
                  <div className="absolute top-2 left-2">
                    <span className="text-[10px] font-bold px-2 py-0.5 rounded-md bg-[#061633]/85 text-blue-300 border border-blue-500/30 backdrop-blur-xs">
                      {item.category}
                    </span>
                  </div>

                  {/* Delete button if user added */}
                  {item.isUserAdded && (
                    <button
                      type="button"
                      onClick={(e) => handleDeletePhoto(item.id, e)}
                      className="absolute top-2 right-2 bg-red-600/90 hover:bg-red-600 text-white rounded-full p-1 shadow-md transition-colors cursor-pointer"
                      title="ลบรูปนี้"
                    >
                      <Trash2 className="w-3.5 h-3.5" />
                    </button>
                  )}
                </div>

                <figcaption className="p-3 text-left">
                  <b className="block text-xs sm:text-sm font-semibold text-slate-200 line-clamp-1">{item.title}</b>
                  <span className="text-[11px] text-slate-400 block line-clamp-1 mt-0.5">{item.desc}</span>
                </figcaption>
              </figure>
            ))}
          </div>

          {filteredGallery.length === 0 && (
            <div className="text-center py-12 bg-[#0C2250]/40 rounded-2xl border border-dashed border-[#1E3A74]">
              <ImageIcon className="w-10 h-10 text-slate-500 mx-auto mb-2" />
              <p className="text-sm text-slate-300">ยังไม่มีรูปภาพในหมวดหมู่นี้</p>
              <button
                type="button"
                onClick={() => {
                  setNewPhotoData(prev => ({ ...prev, category: galleryCategory }));
                  setIsAddPhotoModalOpen(true);
                }}
                className="mt-3 text-xs text-blue-400 hover:underline inline-flex items-center gap-1"
              >
                <Plus className="w-3.5 h-3.5" /> คลิกเพื่อเพิ่มรูปในหมวดนี้
              </button>
            </div>
          )}
        </div>
      </section>

      {/* Lightbox Modal */}
      {lightboxImg && (
        <div 
          onClick={() => setLightboxImg(null)}
          className="fixed inset-0 z-50 bg-[#030A1E]/95 backdrop-blur-md flex items-center justify-center p-4 cursor-zoom-out"
        >
          <div className="relative max-w-3xl max-h-[90vh] flex flex-col items-center">
            <button 
              onClick={() => setLightboxImg(null)}
              className="absolute -top-10 right-0 text-white hover:text-amber-400 transition-colors cursor-pointer"
            >
              <X className="w-7 h-7" />
            </button>
            <img 
              src={lightboxImg.url} 
              alt={lightboxImg.title} 
              className="max-w-full max-h-[80vh] rounded-2xl shadow-2xl border border-[#2B4C94] object-contain"
            />
            <p className="mt-3 text-slate-200 text-sm font-medium text-center">
              {lightboxImg.title}
            </p>
          </div>
        </div>
      )}

      {/* Add New Photo to Gallery Modal */}
      {isAddPhotoModalOpen && (
        <div className="fixed inset-0 z-50 bg-[#030A1E]/80 backdrop-blur-sm flex items-center justify-center p-4 overflow-y-auto">
          <div className="bg-[#0C2250] border border-[#2B4C94] rounded-3xl max-w-md w-full p-6 shadow-2xl relative my-8">
            <button 
              onClick={() => setIsAddPhotoModalOpen(false)}
              className="absolute top-5 right-5 text-slate-400 hover:text-white transition-colors cursor-pointer"
            >
              <X className="w-6 h-6" />
            </button>

            <form onSubmit={handleAddPhotoSubmit} className="space-y-4">
              <div>
                <span className="text-xs text-blue-400 font-bold uppercase tracking-wider block">
                  Add Work Photo
                </span>
                <h4 className="text-xl font-black text-white">
                  เพิ่มรูปภาพผลงานใหม่เข้าสู่เว็บไซต์
                </h4>
                <p className="text-xs text-slate-400 mt-0.5">
                  อัปโหลดรูปภาพงานขนส่งจริงจากมือถือของคุณเพื่อแสดงในแกลเลอรี
                </p>
              </div>

              {/* File Upload Box */}
              <div>
                <label className="block text-xs font-semibold text-slate-300 mb-1">
                  เลือกไฟล์รูปภาพ *
                </label>
                <label className="border-2 border-dashed border-[#2B4C94] hover:border-[#2F80FF] bg-[#16346F]/50 rounded-2xl p-4 flex flex-col items-center justify-center cursor-pointer transition-colors group">
                  <input 
                    type="file" 
                    accept="image/*" 
                    onChange={handlePhotoFileChange} 
                    className="hidden" 
                  />
                  {newPhotoData.imagePreview ? (
                    <div className="relative w-full aspect-video rounded-xl overflow-hidden border border-[#2B4C94]">
                      <img src={newPhotoData.imagePreview} alt="พรีวิวรูปภาพ" className="w-full h-full object-cover" />
                      <span className="absolute bottom-1 right-2 text-[10px] bg-black/70 text-white px-2 py-0.5 rounded">แตะเพื่อเปลี่ยนรูป</span>
                    </div>
                  ) : (
                    <div className="text-center py-4">
                      <Upload className="w-8 h-8 text-blue-400 mx-auto mb-2 group-hover:scale-110 transition-transform" />
                      <span className="text-xs text-slate-200 font-semibold block">แตะที่นี่เพื่อเลือกรูปภาพจากเครื่อง</span>
                      <span className="text-[11px] text-slate-400 block mt-0.5">รองรับไฟล์ JPG, PNG, WEBP</span>
                    </div>
                  )}
                </label>
              </div>

              {/* Photo Title */}
              <div>
                <label className="block text-xs font-semibold text-slate-300 mb-1">
                  ชื่อผลงาน / หัวข้อรูปภาพ *
                </label>
                <input 
                  type="text" 
                  required
                  placeholder="เช่น ย้ายหอพัก นวมินทร์, ขนส่งสินค้าบางพลี"
                  value={newPhotoData.title}
                  onChange={(e) => setNewPhotoData({...newPhotoData, title: e.target.value})}
                  className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                />
              </div>

              {/* Category */}
              <div>
                <label className="block text-xs font-semibold text-slate-300 mb-1">
                  หมวดหมู่ของภาพ
                </label>
                <select
                  value={newPhotoData.category}
                  onChange={(e) => setNewPhotoData({...newPhotoData, category: e.target.value})}
                  className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF] cursor-pointer"
                >
                  <option value="ย้ายหอ/คอนโด">ย้ายหอ/คอนโด</option>
                  <option value="ย้ายบ้าน/ออฟฟิศ">ย้ายบ้าน/ออฟฟิศ</option>
                  <option value="ขนส่งสินค้าโรงงาน">ขนส่งสินค้าโรงงาน</option>
                  <option value="ย้ายมอเตอร์ไซค์บิ๊กไบค์">ย้ายมอเตอร์ไซค์บิ๊กไบค์</option>
                  <option value="รถและอุปกรณ์เซฟตี้">รถและอุปกรณ์เซฟตี้</option>
                </select>
              </div>

              {/* Description */}
              <div>
                <label className="block text-xs font-semibold text-slate-300 mb-1">
                  รายละเอียดเพิ่มเติม (ถ้ามี)
                </label>
                <input 
                  type="text" 
                  placeholder="เช่น ขนเตียง 6 ฟุต ตู้เย็น พร้อมซีนพลาสติกเรียบร้อย"
                  value={newPhotoData.desc}
                  onChange={(e) => setNewPhotoData({...newPhotoData, desc: e.target.value})}
                  className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                />
              </div>

              <div className="flex gap-2.5 pt-2">
                <button
                  type="button"
                  onClick={() => setIsAddPhotoModalOpen(false)}
                  className="flex-1 py-2.5 rounded-xl bg-[#16346F] hover:bg-[#1E3A74] text-slate-300 font-semibold text-xs cursor-pointer"
                >
                  ยกเลิก
                </button>
                <button
                  type="submit"
                  className="flex-1 py-2.5 rounded-xl bg-gradient-to-r from-[#2F80FF] to-blue-700 hover:from-blue-500 hover:to-blue-600 text-white font-bold text-xs shadow-lg shadow-blue-500/25 transition-transform active:scale-95 cursor-pointer"
                >
                  บันทึกรูปภาพ
                </button>
              </div>
            </form>
          </div>
        </div>
      )}

      {/* Tracking Code Lookup Simulation */}
      <section id="tracking" className="py-14 bg-[#030E28]/60 border-y border-[#1E3A74]">
        <div className="max-w-3xl mx-auto px-4">
          <div className="text-center mb-8">
            <span className="text-[#2F80FF] font-bold text-xs tracking-widest uppercase block mb-1">
              Live Job Status
            </span>
            <h3 className="text-2xl sm:text-3xl font-black text-white">
              ตรวจสอบสถานะคิวงาน & พิกัดรถรับจ้าง
            </h3>
            <p className="text-xs sm:text-sm text-slate-400 mt-1">
              กรอกรหัสติดตามงาน JT เพื่อดูสถานะการเดินทางและข้อมูลพนักงานขับรถ
            </p>
          </div>

          <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-6 shadow-xl">
            {/* Search Input */}
            <div className="flex gap-2">
              <div className="relative flex-1">
                <Search className="w-5 h-5 text-slate-400 absolute left-3.5 top-1/2 -translate-y-1/2" />
                <input 
                  type="text" 
                  value={trackingInput}
                  onChange={(e) => setTrackingInput(e.target.value)}
                  placeholder="เช่น JT-2026-8812"
                  className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl pl-11 pr-4 py-3 text-sm text-white placeholder-slate-400 focus:outline-none focus:border-[#2F80FF]"
                />
              </div>
              <button 
                type="button"
                onClick={() => {
                  if (trackingInput.trim()) {
                    showToast(`ค้นหารหัส ${trackingInput} สำเร็จ`);
                  }
                }}
                className="bg-[#2F80FF] hover:bg-blue-600 text-white font-bold px-6 py-3 rounded-xl text-sm transition-colors cursor-pointer"
              >
                ตรวจสอบ
              </button>
            </div>

            {/* Quick Test Codes */}
            <div className="flex items-center gap-2 mt-3 text-xs text-slate-400">
              <span>รหัสตัวอย่าง:</span>
              <button 
                type="button" 
                onClick={() => setTrackingInput('JT-2026-8812')}
                className="text-blue-400 hover:underline cursor-pointer"
              >
                JT-2026-8812
              </button>
              <span>·</span>
              <button 
                type="button" 
                onClick={() => setTrackingInput('JT-2026-9045')}
                className="text-blue-400 hover:underline cursor-pointer"
              >
                JT-2026-9045
              </button>
            </div>

            {/* Tracking Result Card */}
            {activeTrackingResult && (
              <div className="mt-6 pt-6 border-t border-[#1E3A74]">
                <div className="flex flex-col sm:flex-row justify-between sm:items-center gap-2 mb-4">
                  <div>
                    <span className="text-xs text-blue-400 font-bold">รหัสงาน: {activeTrackingResult.code}</span>
                    <h5 className="text-lg font-bold text-white">{activeTrackingResult.status}</h5>
                  </div>
                  <span className="text-xs font-semibold px-3 py-1 rounded-full bg-emerald-500/10 text-emerald-300 border border-emerald-500/30 w-fit">
                    {activeTrackingResult.eta}
                  </span>
                </div>

                {/* Progress Steps */}
                <div className="grid grid-cols-4 gap-2 my-5 text-center text-xs">
                  <div className="space-y-1.5">
                    <div className="w-7 h-7 rounded-full bg-emerald-500 text-slate-950 font-bold mx-auto flex items-center justify-center text-xs">✓</div>
                    <span className="text-slate-300 block text-[11px]">ยืนยันคิวงาน</span>
                  </div>
                  <div className="space-y-1.5">
                    <div className="w-7 h-7 rounded-full bg-blue-500 text-white font-bold mx-auto flex items-center justify-center text-xs animate-pulse">2</div>
                    <span className="text-blue-300 font-semibold block text-[11px]">รถกำลังไปรับ</span>
                  </div>
                  <div className="space-y-1.5">
                    <div className="w-7 h-7 rounded-full bg-[#16346F] text-slate-400 font-bold mx-auto flex items-center justify-center text-xs">3</div>
                    <span className="text-slate-400 block text-[11px]">กำลังเดินทาง</span>
                  </div>
                  <div className="space-y-1.5">
                    <div className="w-7 h-7 rounded-full bg-[#16346F] text-slate-400 font-bold mx-auto flex items-center justify-center text-xs">4</div>
                    <span className="text-slate-400 block text-[11px]">ส่งของสำเร็จ</span>
                  </div>
                </div>

                <div className="bg-[#16346F]/50 rounded-xl p-3.5 text-xs text-slate-300 space-y-1.5 border border-[#2B4C94]">
                  <div><strong>คนขับ:</strong> {activeTrackingResult.driverName}</div>
                  <div><strong>ทะเบียนรถ:</strong> {activeTrackingResult.plate}</div>
                  <div><strong>เส้นทาง:</strong> {activeTrackingResult.route}</div>
                </div>
              </div>
            )}
          </div>
        </div>
      </section>

      {/* Customer Reviews Section */}
      <section className="py-16">
        <div className="max-w-6xl mx-auto px-4">
          <div className="text-center mb-10">
            <span className="text-[#2F80FF] font-bold text-xs tracking-widest uppercase block mb-1">
              Customer Reviews
            </span>
            <h3 className="text-2xl sm:text-3xl font-black text-white">
              เสียงตอบรับจากลูกค้าที่ใช้บริการจริง
            </h3>
            <p className="text-xs sm:text-sm text-slate-400 mt-1">
              ความประทับใจจากลูกค้าขนย้ายหอพัก คอนโด บ้านเดี่ยว และโรงงาน
            </p>
          </div>

          <div className="grid grid-cols-1 md:grid-cols-3 gap-5">
            {[
              {
                name: 'คุณพัชรินทร์ (เตย)',
                place: 'ย้ายคอนโด ลาดพร้าว ➔ อ่อนนุช',
                comment: 'ประทับใจมากค่ะ พี่เจคนขับสุภาพมาก ช่วยซีนพลาสติกเตียงนอน 6 ฟุตกับตู้เย็นให้อย่างดี ไม่มีรอยขีดข่วนเลย ตรงเวลาเป๊ะ ราคาเท่าที่คุยไว้ไม่มีบวกเพิ่ม',
                stars: 5
              },
              {
                name: 'คุณธีรพงศ์ (กอล์ฟ)',
                place: 'ขนส่งสินค้าออกบูธ ไบเทค บางนา',
                comment: 'ใช้บริการส่งของออกบูธด่วนเวลาเช้าตรู่ มาสแตนด์บายก่อนเวลานัด 20 นาที ตู้ทึบสูงใส่โครงสร้างบูธได้พอดีมาก แนะนำเลยครับ ใครหารถรับจ้างที่ไว้ใจได้',
                stars: 5
              },
              {
                name: 'คุณเอกสิทธิ์ (แบงค์)',
                place: 'ย้ายบิ๊กไบค์ Ducati จาก กทม. ➔ ชลบุรี',
                comment: 'ตอนแรกกังวลเรื่องรถบิ๊กไบค์จะเป็นรอย แต่ทาง JT มีสะพานขึ้นอย่างดี รัดสเตย์ 4 จุดแน่นหนามาก ถึงปลายทางอย่างปลอดภัย ขอบคุณมากๆ ครับ',
                stars: 5
              }
            ].map((review, i) => (
              <div key={i} className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-5 shadow-lg flex flex-col justify-between">
                <div>
                  <div className="flex gap-1 text-amber-400 mb-3">
                    {[...Array(review.stars)].map((_, idx) => (
                      <Star key={idx} className="w-4 h-4 fill-amber-400 text-amber-400" />
                    ))}
                  </div>
                  <p className="text-sm text-slate-300 font-light leading-relaxed italic mb-4">
                    "{review.comment}"
                  </p>
                </div>
                <div className="border-t border-[#1E3A74] pt-3">
                  <b className="block text-white text-sm font-semibold">{review.name}</b>
                  <span className="text-[11px] text-blue-400 block">{review.place}</span>
                </div>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* FAQ Accordion Section */}
      <section id="faq" className="py-14 bg-[#030E28]/60 border-t border-[#1E3A74]">
        <div className="max-w-3xl mx-auto px-4">
          <div className="text-center mb-10">
            <span className="text-[#2F80FF] font-bold text-xs tracking-widest uppercase block mb-1">
              FAQ
            </span>
            <h3 className="text-2xl sm:text-3xl font-black text-white">
              คำถามที่พบบ่อย
            </h3>
          </div>

          <div className="space-y-3">
            {faqs.map((faq, index) => (
              <div 
                key={index}
                className="bg-[#0C2250] border border-[#1E3A74] rounded-xl overflow-hidden"
              >
                <button
                  type="button"
                  onClick={() => setActiveFaq(activeFaq === index ? null : index)}
                  className="w-full p-4 text-left font-bold text-sm text-slate-100 flex items-center justify-between gap-3 hover:text-blue-400 transition-colors cursor-pointer"
                >
                  <span className="flex items-center gap-2.5">
                    <HelpCircle className="w-4 h-4 text-blue-400 shrink-0" />
                    {faq.q}
                  </span>
                  <span className="text-xl leading-none text-slate-400">
                    {activeFaq === index ? '−' : '+'}
                  </span>
                </button>
                {activeFaq === index && (
                  <div className="px-4 pb-4 pt-1 text-xs sm:text-sm text-slate-300 font-light leading-relaxed border-t border-[#1E3A74]/50">
                    {faq.a}
                  </div>
                )}
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* Online Booking Modal */}
      {isBookingModalOpen && (
        <div className="fixed inset-0 z-50 bg-[#030A1E]/80 backdrop-blur-sm flex items-center justify-center p-4 overflow-y-auto">
          <div className="bg-[#0C2250] border border-[#2B4C94] rounded-3xl max-w-lg w-full p-6 shadow-2xl relative my-8">
            <button 
              onClick={() => {
                setIsBookingModalOpen(false);
                setBookingConfirmed(false);
              }}
              className="absolute top-5 right-5 text-slate-400 hover:text-white transition-colors cursor-pointer"
            >
              <X className="w-6 h-6" />
            </button>

            {!bookingConfirmed ? (
              <form onSubmit={handleBookingSubmit} className="space-y-4">
                <div>
                  <span className="text-xs text-blue-400 font-bold uppercase tracking-wider block">
                    Online Booking
                  </span>
                  <h4 className="text-xl font-black text-white">
                    จองคิวรถกระบะตู้ทึบออนไลน์
                  </h4>
                  <p className="text-xs text-slate-400 mt-0.5">
                    กรอกข้อมูลเบื้องต้นเพื่อให้เจ้าหน้าที่ล็อกคิวงานและติดต่อกลับทันที
                  </p>
                </div>

                <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">
                  <div>
                    <label className="block text-xs font-semibold text-slate-300 mb-1">
                      ชื่อผู้ติดต่อ *
                    </label>
                    <input 
                      type="text" 
                      required
                      placeholder="เช่น คุณสมชาย"
                      value={bookingForm.customerName}
                      onChange={(e) => setBookingForm({...bookingForm, customerName: e.target.value})}
                      className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                    />
                  </div>

                  <div>
                    <label className="block text-xs font-semibold text-slate-300 mb-1">
                      เบอร์โทรศัพท์ *
                    </label>
                    <input 
                      type="tel" 
                      required
                      placeholder="08X-XXX-XXXX"
                      value={bookingForm.phone}
                      onChange={(e) => setBookingForm({...bookingForm, phone: e.target.value})}
                      className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                    />
                  </div>
                </div>

                <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">
                  <div>
                    <label className="block text-xs font-semibold text-slate-300 mb-1">
                      วันที่ต้องการใช้รถ
                    </label>
                    <input 
                      type="date" 
                      value={bookingForm.serviceDate}
                      onChange={(e) => setBookingForm({...bookingForm, serviceDate: e.target.value})}
                      className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                    />
                  </div>

                  <div>
                    <label className="block text-xs font-semibold text-slate-300 mb-1">
                      เวลาที่ต้องการ
                    </label>
                    <input 
                      type="time" 
                      value={bookingForm.serviceTime}
                      onChange={(e) => setBookingForm({...bookingForm, serviceTime: e.target.value})}
                      className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                    />
                  </div>
                </div>

                <div>
                  <label className="block text-xs font-semibold text-slate-300 mb-1">
                    จุดรับของ (ต้นทาง) *
                  </label>
                  <input 
                    type="text" 
                    required
                    placeholder="เช่น คอนโดย่านลาดพร้าว ซอย 101"
                    value={bookingForm.pickupLocation}
                    onChange={(e) => setBookingForm({...bookingForm, pickupLocation: e.target.value})}
                    className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                  />
                </div>

                <div>
                  <label className="block text-xs font-semibold text-slate-300 mb-1">
                    จุดส่งของ (ปลายทาง)
                  </label>
                  <input 
                    type="text" 
                    placeholder="เช่น ซอยอ่อนนุช 46 หรือ อ.เมือง จ.ชลบุรี"
                    value={bookingForm.dropoffLocation}
                    onChange={(e) => setBookingForm({...bookingForm, dropoffLocation: e.target.value})}
                    className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                  />
                </div>

                <div>
                  <label className="block text-xs font-semibold text-slate-300 mb-1">
                    รายละเอียดสิ่งของที่ต้องการย้าย / หมายเหตุ
                  </label>
                  <textarea 
                    rows={2}
                    placeholder="เช่น เตียง 6 ฟุต, ตู้เย็น 2 ประตู, กล่อง 10 ใบ"
                    value={bookingForm.notes}
                    onChange={(e) => setBookingForm({...bookingForm, notes: e.target.value})}
                    className="w-full bg-[#16346F] border border-[#2B4C94] rounded-xl p-3 text-sm text-white focus:outline-none focus:border-[#2F80FF]"
                  />
                </div>

                {/* Attached Cargo Photos */}
                <div>
                  <div className="flex justify-between items-center mb-1">
                    <label className="text-xs font-semibold text-slate-300 flex items-center gap-1.5">
                      <Camera className="w-3.5 h-3.5 text-blue-400" /> รูปถ่ายสิ่งของ ({cargoPhotos.length} รูป)
                    </label>
                    <label className="text-[11px] text-blue-400 hover:underline cursor-pointer flex items-center gap-1">
                      <Plus className="w-3 h-3" />
                      <span>แนบรูปเพิ่ม</span>
                      <input type="file" multiple accept="image/*" onChange={handleCargoPhotoChange} className="hidden" />
                    </label>
                  </div>
                  {cargoPhotos.length > 0 ? (
                    <div className="flex flex-wrap gap-2 p-2 rounded-xl bg-[#16346F]/40 border border-[#2B4C94]">
                      {cargoPhotos.map((photo, pIdx) => (
                        <div key={pIdx} className="relative w-12 h-12 rounded-lg overflow-hidden border border-[#2B4C94]">
                          <img src={photo} alt="รูปสิ่งของ" className="w-full h-full object-cover" />
                          <button
                            type="button"
                            onClick={() => handleRemoveCargoPhoto(pIdx)}
                            className="absolute top-0 right-0 bg-red-600/90 text-white rounded-bl p-0.5"
                          >
                            <X className="w-2.5 h-2.5" />
                          </button>
                        </div>
                      ))}
                    </div>
                  ) : (
                    <label className="block p-2 rounded-xl border border-dashed border-[#2B4C94] text-center text-[11px] text-slate-400 hover:text-slate-200 cursor-pointer">
                      <span>ยังไม่มีรูปภาพ (แตะเพื่อแนบรูปเตียง, ตู้, โซฟา เพื่อประเมินราคาแม่นยำ)</span>
                      <input type="file" multiple accept="image/*" onChange={handleCargoPhotoChange} className="hidden" />
                    </label>
                  )}
                </div>

                <div className="bg-[#04102A] p-3 rounded-xl border border-[#1E3A74] text-xs flex justify-between items-center">
                  <span className="text-slate-300">ราคาประเมินจากระบบ:</span>
                  <span className="text-base font-bold text-amber-400">
                    {calculation.total.toLocaleString()} บาท
                  </span>
                </div>

                <button
                  type="submit"
                  className="w-full py-3.5 rounded-xl bg-gradient-to-r from-[#2F80FF] to-blue-700 hover:from-blue-500 hover:to-blue-600 text-white font-extrabold text-sm shadow-xl shadow-blue-500/25 transition-transform active:scale-95 cursor-pointer"
                >
                  ยืนยันการจองคิวรถ
                </button>
              </form>
            ) : (
              <div className="text-center py-4 space-y-4">
                <div className="w-16 h-16 rounded-full bg-emerald-500/20 text-emerald-400 flex items-center justify-center mx-auto text-3xl">
                  ✓
                </div>
                <h4 className="text-2xl font-black text-white">
                  จองคิวงานเรียบร้อยแล้ว!
                </h4>
                <div className="bg-[#04102A] p-4 rounded-2xl border border-[#1E3A74] text-left text-xs space-y-2">
                  <div className="flex justify-between">
                    <span className="text-slate-400">รหัสจองงาน:</span>
                    <strong className="text-amber-400 text-sm">{bookingReference}</strong>
                  </div>
                  <div className="flex justify-between">
                    <span className="text-slate-400">ชื่อผู้ติดต่อ:</span>
                    <span className="text-white">{bookingForm.customerName}</span>
                  </div>
                  <div className="flex justify-between">
                    <span className="text-slate-400">เบอร์โทร:</span>
                    <span className="text-white">{bookingForm.phone}</span>
                  </div>
                  <div className="flex justify-between">
                    <span className="text-slate-400">บริการ:</span>
                    <span className="text-white">{calculation.service.name}</span>
                  </div>
                  {cargoPhotos.length > 0 && (
                    <div className="flex justify-between">
                      <span className="text-slate-400">รูปถ่ายสิ่งของ:</span>
                      <span className="text-blue-300 font-semibold">แนบไว้ {cargoPhotos.length} รูป</span>
                    </div>
                  )}
                  <div className="flex justify-between">
                    <span className="text-slate-400">ราคาประเมิน:</span>
                    <span className="text-emerald-400 font-bold">{calculation.total.toLocaleString()} บาท</span>
                  </div>
                </div>

                <p className="text-xs text-slate-300 leading-relaxed">
                  เจ้าหน้าที่กำลังตรวจสอบคิวรถและจะติดต่อกลับเบอร์ <strong>{bookingForm.phone}</strong> ภายใน 3-5 นาทีครับ
                </p>

                <div className="flex flex-col gap-2 pt-2">
                  <a
                    href={`https://line.me/ti/p/~max_milin2564?text=${encodeURIComponent(
                      `สวัสดีครับ แจ้งจองคิวงาน รหัส: ${bookingReference}\nชื่อ: ${bookingForm.customerName}\nเบอร์โทร: ${bookingForm.phone}\nจุดรับ: ${bookingForm.pickupLocation}`
                    )}`}
                    target="_blank"
                    rel="noopener noreferrer"
                    className="w-full py-3 bg-[#00C853] hover:bg-[#00E676] text-slate-950 font-bold rounded-xl text-xs flex items-center justify-center gap-2"
                  >
                    <MessageCircle className="w-4 h-4" />
                    ส่งใบจองนี้เข้า LINE: max_milin2564
                  </a>
                  <button
                    onClick={() => {
                      setIsBookingModalOpen(false);
                      setBookingConfirmed(false);
                    }}
                    className="w-full py-2.5 bg-[#16346F] hover:bg-[#1E3A74] text-slate-300 font-medium rounded-xl text-xs"
                  >
                    ปิดหน้านี้
                  </button>
                </div>
              </div>
            )}
          </div>
        </div>
      )}

      {/* Footer */}
      <footer className="bg-[#031029] border-t border-[#1E3A74] py-12">
        <div className="max-w-6xl mx-auto px-4">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-8 items-center">
            
            {/* Brand & Info */}
            <div>
              <div className="flex items-center gap-3 font-extrabold text-xl text-white mb-2">
                <div className="w-9 h-9 rounded-xl bg-gradient-to-br from-amber-400 to-blue-600 flex items-center justify-center text-slate-950 font-black text-lg">
                  JT
                </div>
                <span>JT TRANSPORT 2026 TH</span>
              </div>
              <p className="text-slate-400 text-xs sm:text-sm font-light leading-relaxed max-w-md">
                บริการรถรับจ้างทั่วไทยส่งสินค้า ย้ายหอ ย้ายบ้าน มอเตอร์ไซค์บิ๊กไบค์ วิ่งตรงทั่วไทย 24 ชั่วโมง ตู้ทึบหลังคาสูง กันน้ำกันฝน 100% สายรัดสเตย์ยึดแน่นหนา ปลอดภัย ไว้ใจได้
              </p>
            </div>

            {/* Contact Direct Card */}
            <div className="bg-[#0C2250] border border-[#1E3A74] rounded-2xl p-5 space-y-2">
              <span className="text-xs font-bold text-[#2F80FF] uppercase tracking-wider block">
                Direct Contact 24/7
              </span>
              <div className="text-sm space-y-1 text-slate-200">
                <div>
                  <strong>เบอร์หลัก:</strong>{' '}
                  <a href="tel:+66879209677" className="text-blue-400 hover:underline font-bold">
                    087-920-9677
                  </a>
                </div>
                <div>
                  <strong>เบอร์สำรอง:</strong>{' '}
                  <a href="tel:+66967744051" className="text-blue-400 hover:underline font-bold">
                    096-774-4051
                  </a>
                </div>
                <div>
                  <strong>LINE ID:</strong>{' '}
                  <a href="https://line.me/ti/p/~max_milin2564" target="_blank" rel="noopener noreferrer" className="text-emerald-400 hover:underline font-bold">
                    max_milin2564
                  </a>
                </div>
              </div>
            </div>

          </div>

          <div className="text-center text-slate-500 text-xs mt-10 pt-6 border-t border-[#1E3A74]/50">
            © 2026 JT TRANSPORT TH. สงวนลิขสิทธิ์ทุกประการ · ให้บริการรถรับจ้างทั่วไทยส่งสินค้า
          </div>
        </div>
      </footer>

      {/* Floating Bottom Action Bar for Mobile */}
      <div className="md:hidden fixed bottom-0 left-0 right-0 z-40 bg-[#061633]/95 backdrop-blur-md border-t border-[#1E3A74] p-2.5 pb-[calc(10px+env(safe-area-inset-bottom,0px))] grid grid-cols-3 gap-2">
        <a 
          href="tel:+66879209677"
          className="flex flex-col items-center justify-center p-2 rounded-xl bg-gradient-to-r from-[#2F80FF] to-blue-700 text-white font-bold text-xs"
        >
          <Phone className="w-4 h-4 mb-0.5" />
          <span>โทรด่วน</span>
        </a>

        <a 
          href="https://line.me/ti/p/~max_milin2564"
          target="_blank"
          rel="noopener noreferrer"
          className="flex flex-col items-center justify-center p-2 rounded-xl bg-[#00C853] text-slate-950 font-bold text-xs"
        >
          <MessageCircle className="w-4 h-4 mb-0.5" />
          <span>แชท LINE</span>
        </a>

        <a 
          href="#calculator"
          className="flex flex-col items-center justify-center p-2 rounded-xl bg-[#0C2250] border border-[#2B4C94] text-slate-200 font-bold text-xs"
        >
          <Calculator className="w-4 h-4 mb-0.5 text-blue-400" />
          <span>เช็คราคา</span>
        </a>
      </div>

    </div>
  );
}
