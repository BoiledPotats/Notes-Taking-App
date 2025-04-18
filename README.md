# Notes-Taking-App
#personalized notes taking app
import { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";

export default function NotesApp() {
  const [notes, setNotes] = useState([]);
  const [newNote, setNewNote] = useState("");
  const [newTag, setNewTag] = useState("");
  const [newFolder, setNewFolder] = useState("");
  const [searchTerm, setSearchTerm] = useState("");
  const [selectedFolder, setSelectedFolder] = useState("All");

  useEffect(() => {
    const savedNotes = localStorage.getItem("notes");
    if (savedNotes) setNotes(JSON.parse(savedNotes));
  }, []);

  useEffect(() => {
    localStorage.setItem("notes", JSON.stringify(notes));
  }, [notes]);

  const addNote = () => {
    if (newNote.trim()) {
      const folder = newFolder.trim() || "General";
      const tag = newTag.trim();
      setNotes((prev) => [
        ...prev,
        {
          id: Date.now(),
          title: `Catatan ${prev.length + 1}`,
          content: newNote,
          expanded: true,
          tag,
          folder,
          images: [],
        },
      ]);
      setNewNote("");
      setNewTag("");
      setNewFolder("");
    }
  };

  const toggleExpand = (id) => {
    setNotes((prev) =>
      prev.map((note) =>
        note.id === id ? { ...note, expanded: !note.expanded } : note
      )
    );
  };

  const updateContent = (id, content) => {
    setNotes((prev) =>
      prev.map((note) => (note.id === id ? { ...note, content } : note))
    );
  };

  const deleteNote = (id) => {
    setNotes((prev) => prev.filter((note) => note.id !== id));
  };

  const filteredNotes = notes.filter(
    (note) =>
      note.content.toLowerCase().includes(searchTerm.toLowerCase()) &&
      (selectedFolder === "All" || note.folder === selectedFolder)
  );

  const uploadImage = (id, file) => {
    const reader = new FileReader();
    reader.onload = () => {
      setNotes((prev) =>
        prev.map((note) =>
          note.id === id
            ? { ...note, images: [...note.images, reader.result] }
            : note
        )
      );
    };
    reader.readAsDataURL(file);
  };

  const uniqueFolders = ["All", ...new Set(notes.map((note) => note.folder))];

  return (
    <div className="max-w-2xl mx-auto p-4 space-y-4">
      <h1 className="text-2xl font-bold">Notes App</h1>

      <div className="grid grid-cols-1 sm:grid-cols-2 gap-2">
        <Input
          placeholder="Tulis isi catatan..."
          value={newNote}
          onChange={(e) => setNewNote(e.target.value)}
        />
        <Input
          placeholder="Folder (opsional)"
          value={newFolder}
          onChange={(e) => setNewFolder(e.target.value)}
        />
        <Input
          placeholder="Tag (opsional)"
          value={newTag}
          onChange={(e) => setNewTag(e.target.value)}
        />
        <Button onClick={addNote}>Tambah Catatan</Button>
      </div>

      <div className="flex gap-2 items-center">
        <Input
          placeholder="Cari catatan..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
        <select
          className="border rounded px-2 py-1"
          value={selectedFolder}
          onChange={(e) => setSelectedFolder(e.target.value)}
        >
          {uniqueFolders.map((folder) => (
            <option key={folder} value={folder}>
              {folder}
            </option>
          ))}
        </select>
      </div>

      <div className="space-y-2">
        {filteredNotes.map((note) => (
          <Card key={note.id} className="rounded-2xl shadow">
            <CardContent className="p-4">
              <div className="flex justify-between items-center">
                <h2
                  className="text-lg font-semibold cursor-pointer"
                  onClick={() => toggleExpand(note.id)}
                >
                  {note.title} {note.tag && <span className="text-xs text-gray-500">#{note.tag}</span>}
                </h2>
                <div className="flex gap-2">
                  <Button size="sm" variant="outline" onClick={() => toggleExpand(note.id)}>
                    {note.expanded ? "Tutup" : "Buka"}
                  </Button>
                  <Button size="sm" variant="destructive" onClick={() => deleteNote(note.id)}>
                    Hapus
                  </Button>
                </div>
              </div>
              {note.expanded && (
                <div className="mt-2 space-y-2">
                  <Textarea
                    value={note.content}
                    onChange={(e) => updateContent(note.id, e.target.value)}
                    className="w-full"
                  />
                  <input
                    type="file"
                    accept="image/*"
                    onChange={(e) => e.target.files && uploadImage(note.id, e.target.files[0])}
                  />
                  <div className="flex flex-wrap gap-2 mt-2">
                    {note.images.map((src, idx) => (
                      <img
                        key={idx}
                        src={src}
                        alt="note-img"
                        className="h-24 w-24 object-cover rounded"
                      />
                    ))}
                  </div>
                </div>
              )}
            </CardContent>
          </Card>
        ))}
      </div>
    </div>
  );
}
