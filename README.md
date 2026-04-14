# modules-index-controllers-home.php
<?php
/**
 * @filesource modules/index/controllers/home.php
 * @link http://kotchasan.com
 * @copyright 2016 Goragod.com
 * @license http://kotchasan.comlicense/
 */

namespace Index\Home;

use \Kotchasan\Http\Request;
use \Kotchasan\Html;
use \Kotchasan\Login;

/**
 * Controller สำหรับหน้า Dashboard
 */
class Controller extends \Gcms\Controller
{
    public function render(Request $request)
    {
        // ตรวจสอบการ Login
        $login = Login::isMember();
        if ($login) {
            // กำหนดค่าเริ่มต้นของหน้า
            $this->title = '{LNG_Dashboard}';
            $this->menu = 'home';
            
            // สร้างคอนเทนเนอร์หลัก
            $section = Html::create('section', [
                'class' => 'content_section'
            ]);

            // ส่วนหัวของหน้า
            $section->add('header', [
                'innerHTML' => '<h1><span class="icon-home">{LNG_Dashboard}</span></h1>'
            ]);

            // 1. เตรียมข้อมูลการ์ด (Cards)
            $card = new \Kotchasan\Accordion();

            // --- ส่วนที่เพิ่มใหม่: เช็คจำนวนคนรอรีเซ็ตรหัสผ่าน (เฉพาะ Admin) ---
            if ($login['status'] == 1) {
                $reset_count = \Kotchasan\Model::createQuery()
                    ->selectCount()
                    ->from('user')
                    ->where(['active', 0])
                    ->executeCount();

                $card->add('reset_password', [
                    'title' => 'คำขอรหัสผ่านใหม่',
                    'value' => $reset_count,
                    'url' => 'index.php?module=member&sort=active%20asc',
                    'class' => 'icon-unlocked color-pink'
                ]);
            }

            // --- ส่วนที่เพิ่มใหม่: การ์ดแก้ไขโปรไฟล์ตัวเอง ---
            $card->add('profile', [
                'title' => '{LNG_My Profile}',
                'value' => 'Update',
                'url' => 'index.php?module=member-setup&id='.$login['id'],
                'class' => 'icon-user-address color-emerald'
            ]);

            // แสดงการ์ดที่หน้าจอ
            $section->add('div', [
                'class' => 'card-section',
                'innerHTML' => $card->render()
            ]);

            // 2. เตรียมข้อมูลเมนูทางลัด (Quick Menu)
            $menu = new \Kotchasan\Accordion();

            // --- ส่วนที่เพิ่มใหม่: ปุ่มเปลี่ยนรหัสผ่าน ---
            $menu->add('password', [
                'title' => '{LNG_Change Password}',
                'url' => 'index.php?module=password',
                'class' => 'icon-key color-crimson'
            ]);

            // ถ้ามีเมนูอื่นๆ ในระบบ ให้ดึงมาแสดงด้วย (โค้ดมาตรฐานเดิม)
            foreach (\Gcms\Gcms::$module->getControllers('addMenu') as $className) {
                $className::addMenu($menu, $login);
            }

            // แสดงเมนูทางลัดที่หน้าจอ
            $section->add('div', [
                'class' => 'quick-menu-section',
                'innerHTML' => $menu->render()
            ]);

            return $section->render();
        }
        
        // ถ้าไม่ได้ Login ให้ส่งไปหน้า Login
        return \Index\Error\Controller::execute($this, $request->getUri());
    }
}
