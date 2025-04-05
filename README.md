import { useState } from "react";
import { Textarea } from "@/components/ui/textarea";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Loader2, BookOpenCheck, Book, UserCheck } from "lucide-react";

export default function SkillSqueeze() {
  const [prompt, setPrompt] = useState("");
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState("");
  const [paid, setPaid] = useState(false);
  const [userHistory, setUserHistory] = useState([]);
  const [plan, setPlan] = useState("free");

  const handleGenerate = async () => {
    if (!prompt.trim()) return;
    setLoading(true);
    setResult("");
    try {
      const res = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer YOUR_OPENAI_API_KEY`,
        },
        body: JSON.stringify({
          model: "gpt-4",
          messages: [
            {
              role: "system",
              content: "You are a helpful assistant that generates detailed digital products like courses, ebooks, and guides from user prompts."
            },
            {
              role: "user",
              content: prompt
            }
          ],
          temperature: 0.7
        })
      });
      const data = await res.json();
      setResult(data.choices[0].message.content);
      setUserHistory(prev => [...prev, { prompt, content: data.choices[0].message.content }]);
    } catch (err) {
      setResult("Error generating content.");
    } finally {
      setLoading(false);
    }
  };

  const handleDownloadPDF = () => {
    const element = document.createElement("a");
    const file = new Blob([result], { type: 'application/pdf' });
    element.href = URL.createObjectURL(file);
    element.download = "skillsqueeze_output.pdf";
    document.body.appendChild(element);
    element.click();
  };

  const handlePaystack = () => {
    const handler = window.PaystackPop.setup({
      key: 'YOUR_PAYSTACK_PUBLIC_KEY',
      email: 'customer@email.com',
      amount: plan === "pro" ? 5000 * 100 : 1000 * 100, // Adjust pricing
      currency: 'ZAR',
      callback: function (response) {
        setPaid(true);
        alert('Payment complete! Transaction ref is ' + response.reference);
      },
      onClose: function () {
        alert('Payment window closed');
      }
    });
    handler.openIframe();
  };

  return (
    <div className="max-w-2xl mx-auto py-12 px-4 text-center bg-gradient-to-br from-black via-zinc-900 to-gray-800 text-white rounded-2xl shadow-2xl border border-zinc-700">
      <h1 className="text-4xl font-bold mb-4 text-green-400">SkillSqueeze</h1>
      <p className="text-lg text-zinc-300 mb-6">
        Instantly generate eBooks, digital courses, and guides from a single powerful prompt.
      </p>

      {!paid ? (
        <>
          <div className="flex flex-col space-y-3 mb-4">
            <select
              className="bg-zinc-800 text-white p-2 rounded border border-zinc-600"
              value={plan}
              onChange={(e) => setPlan(e.target.value)}
            >
              <option value="free">Free Trial (1 use)</option>
              <option value="basic">Basic - R10</option>
              <option value="pro">Pro - R50</option>
            </select>
            <Button className="bg-green-500 hover:bg-green-600 text-white text-lg py-6" onClick={handlePaystack}>
              Unlock {plan.charAt(0).toUpperCase() + plan.slice(1)} Access
            </Button>
            <p className="text-sm text-muted-foreground text-zinc-400">
              Secure Paystack Checkout • Instant Access
            </p>
          </div>
        </>
      ) : (
        <>
          <Card className="mb-6 bg-zinc-800 border border-zinc-700">
            <CardContent className="p-4">
              <Textarea
                value={prompt}
                onChange={(e) => setPrompt(e.target.value)}
                placeholder="e.g. How to build confidence, Learn crypto in 7 days..."
                className="min-h-[120px] bg-zinc-900 text-white"
              />
              <Button onClick={handleGenerate} className="mt-4 w-full bg-green-500 hover:bg-green-600 text-white" disabled={loading}>
                {loading ? <Loader2 className="animate-spin h-4 w-4 mr-2" /> : <BookOpenCheck className="h-4 w-4 mr-2" />}
                Generate
              </Button>
            </CardContent>
          </Card>

          {result && (
            <div className="text-left bg-zinc-900 p-4 rounded-xl shadow text-zinc-100 border border-zinc-700">
              <h2 className="text-xl font-semibold mb-2 flex items-center text-green-400">
                <Book className="mr-2 h-5 w-5" /> Your Generated Content
              </h2>
              <pre className="whitespace-pre-wrap font-sans text-sm text-zinc-300">
                {result}
              </pre>
              <Button onClick={handleDownloadPDF} className="mt-4 bg-green-600 hover:bg-green-700 text-white">
                Download as PDF
              </Button>
            </div>
          )}

          {userHistory.length > 0 && (
            <div className="mt-10 text-left">
              <h3 className="text-lg text-green-300 font-semibold mb-2 flex items-center">
                <UserCheck className="h-4 w-4 mr-2" /> Your History
              </h3>
              <ul className="list-disc ml-5 text-sm text-zinc-400">
                {userHistory.map((item, idx) => (
                  <li key={idx} className="mb-1">
                    <strong>Prompt:</strong> {item.prompt.substring(0, 50)}...
                  </li>
                ))}
              </ul>
            </div>
          )}
        </>
      )}
    </div>
  );
}
