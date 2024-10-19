## Copy this code base


import { useEffect, useState } from "react";
import type { Schema } from "../amplify/data/resource";
import { useAuthenticator } from '@aws-amplify/ui-react';
import { generateClient } from "aws-amplify/data";

const client = generateClient<Schema>();

function App() {
  const [todos, setTodos] = useState<Array<Schema["Todo"]["type"]>>([]);
  const [newTodoContent, setNewTodoContent] = useState(""); 
  const [isFormVisible, setIsFormVisible] = useState(false); 
  const { user } = useAuthenticator();

  useEffect(() => {
    client.models.Todo.observeQuery().subscribe({
      next: (data) => setTodos([...data.items]),
    });
  }, []);

  async function createTodo(e: React.FormEvent) {
    e.preventDefault(); 
    if (newTodoContent.trim()) {
      await client.models.Todo.create({ content: newTodoContent });
      setNewTodoContent(""); 
      setIsFormVisible(false); 
    }
  }

  function getUsernameFromEmail(email: string) {
    email = email.split('@')[0];
    return email.charAt(0).toUpperCase() + email.slice(1);
  }  

  return (
    <main>
      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: '1rem' }}>
        <h1>{getUsernameFromEmail(user?.signInDetails?.loginId)}'s todos</h1>
      </div>

      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
        <h1>My todos</h1>
        <button onClick={() => setIsFormVisible(!isFormVisible)}>
          {isFormVisible ? 'Hide Form' : '+'} 
        </button> 
      </div>

      {isFormVisible && (
        <form onSubmit={createTodo} style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '1rem' }}>
          <input 
            type="text" 
            placeholder="Enter new todo..." 
            value={newTodoContent} 
            onChange={(e) => setNewTodoContent(e.target.value)} 
            style={{ marginRight: '0.5rem' }}
          />
          <button style={{ backgroundColor: 'blue'}} type="submit">Save Todo</button>
        </form>
      )}

      <ul>
        {todos.map((todo) => (
          <li
            style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: "0.5rem" }}
            key={todo.id}
          >
            {todo.content}
          </li>
        ))}
      </ul>
    </main>
  );
}

export default App;
