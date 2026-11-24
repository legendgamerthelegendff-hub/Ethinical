from http.server import HTTPServer, BaseHTTPRequestHandler
import json

# Simulated session store
sessions = {}

class SessionFixationHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # Check if client already has a session ID in URL or cookie
        path = self.path
        if 'session_id' in path:
            # Extract session ID from URL
            sid = path.split('session_id=')[1].split('&')[0]
            self.send_response(200)
            self.send_header('Content-Type', 'text/html')
            self.end_headers()
            
            html = f"""
            <html>
            <body>
                <h1>Welcome! Your Session ID is: {sid}</h1>
                <p><a href="/login?session_id={sid}">Click here to login</a></p>
            </body>
            </html>
            """
            self.wfile.write(html.encode())
        elif '/login' in path:
            # Simulate login process
            sid = path.split('session_id=')[1]
            sessions[sid] = {'user': 'victim', 'authenticated': True}
            
            self.send_response(200)
            self.send_header('Content-Type', 'text/html')
            self.end_headers()
            
            html = f"""
            <html>
            <body>
                <h1>Login Successful!</h1>
                <p>Session ID: {sid}</p>
                <p><a href="/dashboard?session_id={sid}">Go to Dashboard</a></p>
            </body>
            </html>
            """
            self.wfile.write(html.encode())
        elif '/dashboard' in path:
            sid = path.split('session_id=')[1]
            if sessions.get(sid, {}).get('authenticated'):
                self.send_response(200)
                self.send_header('Content-Type', 'application/json')
                self.end_headers()
                # Return sensitive session details
                response = json.dumps({
                    "status": "success",
                    "user": sessions[sid]['user'],
                    "session_id": sid,
                    "data": "Confidential User Details"
                })
                self.wfile.write(response.encode())
            else:
                self.send_response(401)
                self.end_headers()

    def log_message(self, format, *args):
        pass  # Suppress default logging

if __name__ == '__main__':
    server = HTTPServer(('localhost', 8080), SessionFixationHandler)
    print("Server running on http://localhost:8080")
    server.serve_forever()
