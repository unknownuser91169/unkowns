const http = require("http");

const data = {};

const server = http.createServer((req, res) => {
  res.setHeader("Content-Type", "application/json");
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Methods", "POST, OPTIONS");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type");

  if (req.method === "OPTIONS") {
    res.writeHead(204);
    return res.end();
  }

  if (req.url !== "/rpc" || req.method !== "POST") {
    res.writeHead(404);
    return res.end(JSON.stringify({ error: "Not found" }));
  }

  let body = "";

  req.on("data", chunk => body += chunk);
  req.on("end", () => {
    try {
      const rpc = JSON.parse(body);
      let result;

      if (rpc.method === "setData") {
        data[rpc.params.key] = rpc.params.value;
        result = { success: true };
      } else if (rpc.method === "getData") {
        result = data[rpc.params.key] ?? null;
      } else {
        throw new Error("Method not found");
      }

      res.end(JSON.stringify({
        jsonrpc: "2.0",
        result,
        id: rpc.id
      }));
    } catch (error) {
      res.end(JSON.stringify({
        jsonrpc: "2.0",
        error: { code: -32603, message: error.message },
        id: null
      }));
    }
  });
});

server.listen(process.env.PORT || 3000, "0.0.0.0");
