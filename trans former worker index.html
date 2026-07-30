/**
 * Mega Prompt — Cloudflare Worker backend
 * ------------------------------------------------------------------
 * Handles three things, all under ONE Worker:
 *   1. AI photo transformation (proxies to fal.ai, keeps FAL_KEY secret)
 *   2. Real persistent storage for products & orders (via Cloudflare KV)
 *   3. Real server-side admin authentication (ADMIN_PASSWORD secret,
 *      never shipped to the browser — fixes the "visible in View Source"
 *      problem from the old client-side-only check)
 *
 * SETUP (in addition to the FAL_KEY secret from before):
 * 1. Go to your Worker -> Settings -> Bindings -> Add -> KV Namespace
 *      Variable name:  MEGA_KV
 *      KV namespace:   mega_prompt_data   (id: 5954627f90614f3abfd89a40ecef708f)
 * 2. Go to Settings -> Variables and Secrets -> Add secret:
 *      name:  ADMIN_PASSWORD
 *      value: <choose a real admin password — do NOT reuse the old one,
 *              since it was exposed in the old file's source code>
 * 3. Deploy.
 *
 * Routes (all under your Worker's base URL, e.g. https://xxx.workers.dev):
 *   POST /transform        { image, prompt, negativePrompt? }        -> { imageUrl }
 *   GET  /products                                                    -> { value }  (public read)
 *   POST /products          { password, value }                       -> { ok }      (admin write)
 *   POST /orders/create      { productId, productTitle, price, phone, appUsed, ref } -> { code }
 *   GET  /orders/track?code=XXX                                       -> { status, productId }
 *   GET  /orders/list?password=XXX                                    -> { value }  (admin only)
 *   POST /orders/approve     { password, code }                       -> { ok }
 *   POST /admin/verify       { password }                             -> { ok }
 */

const CORS_HEADERS = {
  "Access-Control-Allow-Origin": "*", // tighten to your real domain once you have one
  "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type",
};

function json(obj, status = 200) {
  return new Response(JSON.stringify(obj), {
    status,
    headers: { ...CORS_HEADERS, "Content-Type": "application/json" },
  });
}

function checkAdminPassword(env, password) {
  return !!env.ADMIN_PASSWORD && password === env.ADMIN_PASSWORD;
}

function genCode() {
  return Math.random().toString(36).slice(2, 8).toUpperCase();
}

export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    const path = url.pathname;

    if (request.method === "OPTIONS") {
      return new Response(null, { headers: CORS_HEADERS });
    }

    if (!env.MEGA_KV && path !== "/transform") {
      return json({ error: "Server misconfigured: MEGA_KV binding is not set (see setup notes in this file)" }, 500);
    }

    // ---------- Admin login ----------
    if (path === "/admin/verify" && request.method === "POST") {
      const body = await safeJson(request);
      if (checkAdminPassword(env, body?.password)) return json({ ok: true });
      return json({ ok: false, error: "Incorrect password" }, 401);
    }

    // ---------- Products ----------
    if (path === "/products" && request.method === "GET") {
      const value = await env.MEGA_KV.get("products");
      return json({ value: value || null });
    }
    if (path === "/products" && request.method === "POST") {
      const body = await safeJson(request);
      if (!checkAdminPassword(env, body?.password)) return json({ error: "Incorrect password" }, 401);
      await env.MEGA_KV.put("products", JSON.stringify(body.value));
      return json({ ok: true });
    }

    // ---------- Orders ----------
    if (path === "/orders/create" && request.method === "POST") {
      const body = await safeJson(request);
      if (!body?.productId || !body?.phone) return json({ error: "Missing required order fields" }, 400);
      const raw = await env.MEGA_KV.get("orders");
      const orders = raw ? JSON.parse(raw) : [];
      const code = genCode();
      orders.push({
        code,
        productId: body.productId,
        productTitle: body.productTitle || "",
        price: body.price || 0,
        phone: body.phone,
        appUsed: body.appUsed || "",
        ref: body.ref || "",
        status: "pending",
        createdAt: Date.now(),
      });
      await env.MEGA_KV.put("orders", JSON.stringify(orders));
      return json({ code });
    }

    if (path === "/orders/track" && request.method === "GET") {
      const code = url.searchParams.get("code");
      const raw = await env.MEGA_KV.get("orders");
      const orders = raw ? JSON.parse(raw) : [];
      const order = orders.find(o => o.code === code);
      if (!order) return json({ error: "Order not found" }, 404);
      return json({ status: order.status, productId: order.productId });
    }

    if (path === "/orders/list" && request.method === "GET") {
      const password = url.searchParams.get("password");
      if (!checkAdminPassword(env, password)) return json({ error: "Incorrect password" }, 401);
      const raw = await env.MEGA_KV.get("orders");
      return json({ value: raw || null });
    }

    if (path === "/orders/approve" && request.method === "POST") {
      const body = await safeJson(request);
      if (!checkAdminPassword(env, body?.password)) return json({ error: "Incorrect password" }, 401);
      const raw = await env.MEGA_KV.get("orders");
      const orders = raw ? JSON.parse(raw) : [];
      const order = orders.find(o => o.code === body.code);
      if (!order) return json({ error: "Order not found" }, 404);
      order.status = "approved";
      await env.MEGA_KV.put("orders", JSON.stringify(orders));
      return json({ ok: true });
    }

    // ---------- AI photo transform (unchanged from before, moved to /transform) ----------
    if (path === "/transform" && request.method === "POST") {
      const body = await safeJson(request);
      const { image, prompt, negativePrompt } = body || {};
      if (!image || !prompt) return json({ error: "Both 'image' and 'prompt' are required" }, 400);
      if (!env.FAL_KEY) return json({ error: "Server misconfigured: FAL_KEY secret is not set" }, 500);

      try {
        const falBody = { prompt, image_urls: [image] };
        if (negativePrompt) falBody.negative_prompt = negativePrompt;

        const falRes = await fetch("https://fal.run/fal-ai/nano-banana-2/edit", {
          method: "POST",
          headers: { "Authorization": `Key ${env.FAL_KEY}`, "Content-Type": "application/json" },
          body: JSON.stringify(falBody),
        });

        if (!falRes.ok) {
          const detail = await falRes.text();
          return json({ error: "Image generation request failed", detail }, 502);
        }
        const data = await falRes.json();
        const outputUrl = data?.images?.[0]?.url;
        if (!outputUrl) return json({ error: "No image returned from the generation service", detail: data }, 502);
        return json({ imageUrl: outputUrl });
      } catch (err) {
        return json({ error: "Unexpected server error", detail: String(err) }, 500);
      }
    }

    return json({ error: "Not found", path }, 404);
  },
};

async function safeJson(request) {
  try { return await request.json(); } catch { return null; }
}
