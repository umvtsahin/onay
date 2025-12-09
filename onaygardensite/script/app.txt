/* app.js - modernized: mobile menu, lightbox with nav, form validation, active menu */
document.addEventListener("DOMContentLoaded", () => {

  // ===== page fade-in
  document.documentElement.classList.add("js-enabled");

  // ===== sticky shadow on scroll
  const navbar = document.querySelector(".navbar");
  window.addEventListener("scroll", () => {
    if (window.scrollY > 30) navbar.classList.add("scrolled");
    else navbar.classList.remove("scrolled");
  });

  // ===== active link highlight (based on filename)
  const current = (window.location.pathname.split("/").pop() || "index.html");
  document.querySelectorAll(".primary-menu a").forEach(a => {
    if (a.getAttribute("href") === current) a.classList.add("active");
  });

  // ===== mobile menu toggle + accessibility
  const hamburger = document.querySelectorAll(".hamburger");
  const menu = document.querySelectorAll(".primary-menu");
  const body = document.body;
  hamburger.forEach(btn => {
    btn.addEventListener("click", () => {
      const expanded = btn.getAttribute("aria-expanded") === "true";
      btn.setAttribute("aria-expanded", String(!expanded));
      // toggle the first .primary-menu (all pages have one)
      const pm = document.querySelector(".primary-menu");
      if (!pm) return;
      pm.classList.toggle("open");
      // lock scroll
      body.classList.toggle("no-scroll");
    });
  });

  // close menu when clicking link (mobile)
  document.querySelectorAll(".primary-menu a").forEach(link => {
    link.addEventListener("click", () => {
      const pm = document.querySelector(".primary-menu");
      if (pm && pm.classList.contains("open")) {
        pm.classList.remove("open");
        const hb = document.querySelector(".hamburger");
        if (hb) hb.setAttribute("aria-expanded", "false");
        body.classList.remove("no-scroll");
      }
    });
  });

  // ===== gallery lightbox with navigation
  const galleryImgs = Array.from(document.querySelectorAll(".gallery-grid img"));
  if (galleryImgs.length) {
    let currentIndex = 0;
    const createLightbox = (idx) => {
      currentIndex = idx;
      const src = galleryImgs[idx].getAttribute("src");
      const alt = galleryImgs[idx].getAttribute("alt") || "";

      // wrapper
      let lb = document.querySelector(".lightbox");
      if (!lb) {
        lb = document.createElement("div");
        lb.className = "lightbox";
        document.body.appendChild(lb);
      }
      lb.innerHTML = `
        <button class="lb-close" aria-label="Kapat">✕</button>
        <div class="lb-frame">
          <img src="${src}" alt="${alt}">
        </div>
        <div class="lb-controls">
          <button class="lb-prev" aria-label="Önceki">‹</button>
          <button class="lb-next" aria-label="Sonraki">›</button>
        </div>
      `;
      // show
      requestAnimationFrame(()=> lb.classList.add("open"));

      // events
      lb.querySelector(".lb-close").addEventListener("click", closeLightbox);
      lb.querySelector(".lb-prev").addEventListener("click", showPrev);
      lb.querySelector(".lb-next").addEventListener("click", showNext);
      lb.addEventListener("click", (e) => {
        if (e.target === lb) closeLightbox();
      });

      // keyboard
      document.addEventListener("keydown", kbHandler);
    };

    const closeLightbox = () => {
      const lb = document.querySelector(".lightbox");
      if (!lb) return;
      lb.classList.remove("open");
      setTimeout(()=> lb.remove(), 320);
      document.removeEventListener("keydown", kbHandler);
    };

    const showPrev = () => {
      currentIndex = (currentIndex - 1 + galleryImgs.length) % galleryImgs.length;
      updateLightboxSrc(currentIndex);
    };
    const showNext = () => {
      currentIndex = (currentIndex + 1) % galleryImgs.length;
      updateLightboxSrc(currentIndex);
    };
    const updateLightboxSrc = (idx) => {
      const lb = document.querySelector(".lightbox");
      if (!lb) return;
      const img = lb.querySelector(".lb-frame img");
      img.src = galleryImgs[idx].getAttribute("src");
      img.alt = galleryImgs[idx].getAttribute("alt") || "";
    };
    const kbHandler = (e) => {
      if (e.key === "Escape") closeLightbox();
      if (e.key === "ArrowLeft") showPrev();
      if (e.key === "ArrowRight") showNext();
    };

    galleryImgs.forEach((imgEl, idx) => {
      imgEl.addEventListener("click", (e) => {
        createLightbox(idx);
      });
    });
  }

  // ===== simple contact form handling (local demo)
  const contactForm = document.getElementById("contactForm");
  if (contactForm) {
    contactForm.addEventListener("submit", (e) => {
      e.preventDefault();
      const name = contactForm.querySelector("#name");
      const email = contactForm.querySelector("#email");
      const message = contactForm.querySelector("#message");
      const msgEl = document.getElementById("formMsg");

      // basic validation
      if (!name.value.trim() || !email.value.trim() || !message.value.trim()) {
        msgEl.textContent = "Lütfen tüm alanları doldurun.";
        msgEl.style.color = "#b44";
        return;
      }
      // basic email check
      if (!/^\S+@\S+\.\S+$/.test(email.value)) {
        msgEl.textContent = "Geçerli bir e-posta girin.";
        msgEl.style.color = "#b44";
        return;
      }

      // demo success (replace with actual API call)
      msgEl.textContent = "Mesajınız gönderildi. Teşekkürler!";
      msgEl.style.color = "#2a7a2a";
      contactForm.reset();

      // optionally, send to server with fetch here
      // fetch('/api/contact', { method:'POST', body: new FormData(contactForm) })...
    });
  }

  // ===== accessibility: focus outline for keyboard users
  document.body.addEventListener('keydown', (e) => {
    if (e.key === 'Tab') document.documentElement.classList.add('show-focus');
  });

});
