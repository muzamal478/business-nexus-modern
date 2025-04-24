## Week 2: Dashboards + Profiles + Collaboration System

### Update `db.json` (Add More Data)
```json
{
  "users": [
    { "id": 1, "email": "investor@example.com", "password": "pass123", "role": "investor", "name": "John Investor" },
    { "id": 2, "email": "entrepreneur@example.com", "password": "pass123", "role": "entrepreneur", "name": "Jane Entrepreneur" },
    { "id": 3, "email": "investor2@example.com", "password": "pass123", "role": "investor", "name": "Mike Investor" }
  ],
  "entrepreneurs": [
    { "id": 1, "name": "Jane Entrepreneur", "startup": "TechTrend", "pitch": "Innovative AI solutions", "bio": "Passionate founder", "funding": 500000, "pitchDeck": "deck.pdf" },
    { "id": 2, "name": "Sam Startup", "startup": "GreenEnergy", "pitch": "Sustainable energy solutions", "bio": "Eco-warrior", "funding": 750000, "pitchDeck": "deck2.pdf" }
  ],
  "investors": [
    { "id": 1, "name": "John Investor", "bio": "Experienced VC", "interests": ["AI", "Fintech"], "portfolio": ["CompanyA", "CompanyB"] },
    { "id": 2, "name": "Mike Investor", "bio": "Angel investor", "interests": ["Healthtech", "Edtech"], "portfolio": ["CompanyC"] }
  ],
  "requests": [
    { "id": 1, "investorId": 1, "entrepreneurId": 1, "status": "pending" },
    { "id": 2, "investorId": 2, "entrepreneurId": 1, "status": "accepted" }
  ]
}
```

### Code: Week 2

#### `src/pages/dashboard/InvestorDashboard.jsx`
```jsx
import { useEffect, useState } from "react";
import { Link } from "react-router-dom";
import api from "../../services/api";
import Card from "../../components/common/Card";
import Button from "../../components/common/Button";

function InvestorDashboard() {
  const [entrepreneurs, setEntrepreneurs] = useState([]);

  useEffect(() => {
    api.get("/entrepreneurs").then((response) => {
      setEntrepreneurs(response.data);
    });
  }, []);

  return (
    <div>
      <h2 className="text-2xl mb-4">Entrepreneurs</h2>
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {entrepreneurs.map((entrepreneur) => (
          <Card key={entrepreneur.id}>
            <h3 className="text-lg font-bold">{entrepreneur.name}</h3>
            <p className="text-sm text-gray-600">{entrepreneur.startup}</p>
            <p className="mt-2">{entrepreneur.pitch}</p>
            <Link to={`/profile/entrepreneur/${entrepreneur.id}`}>
              <Button className="mt-4">View Full Profile</Button>
            </Link>
          </Card>
        ))}
      </div>
    </div>
  );
}

export default InvestorDashboard;
```

#### `src/pages/dashboard/EntrepreneurDashboard.jsx`
```jsx
import { useEffect, useState } from "react";
import api from "../../services/api";
import Card from "../../components/common/Card";
import Button from "../../components/common/Button";

function EntrepreneurDashboard() {
  const [requests, setRequests] = useState([]);

  useEffect(() => {
    api.get("/requests").then(async (response) => {
      const requestsWithInvestor = await Promise.all(
        response.data.map(async (req) => {
          const investor = await api.get(`/investors/${req.investorId}`);
          return { ...req, investor: investor.data };
        })
      );
      setRequests(requestsWithInvestor);
    });
  }, []);

  return (
    <div>
      <h2 className="text-2xl mb-4">Collaboration Requests</h2>
      <div className="space-y-4">
        {requests.map((request) => (
          <Card key={request.id}>
            <h3 className="text-lg font-bold">{request.investor.name}</h3>
            <p className="text-sm text-gray-600">{request.investor.bio}</p>
            <p className="mt-2">Status: {request.status}</p>
            <Button className="mt-4">Respond</Button>
          </Card>
        ))}
      </div>
    </div>
  );
}

export default EntrepreneurDashboard;
```

#### `src/pages/profile/InvestorProfile.jsx`
```jsx
import { useEffect, useState } from "react";
import { useParams } from "react-router-dom";
import api from "../../services/api";
import Card from "../../components/common/Card";

function InvestorProfile() {
  const { id } = useParams();
  const [investor, setInvestor] = useState(null);

  useEffect(() => {
    api.get(`/investors/${id}`).then((response) => {
      setInvestor(response.data);
    });
  }, [id]);

  if (!investor) return <div>Loading...</div>;

  return (
    <Card>
      <h2 className="text-2xl mb-4">{investor.name}</h2>
      <p><strong>Bio:</strong> {investor.bio}</p>
      <p><strong>Interests:</strong> {investor.interests.join(", ")}</p>
      <p><strong>Portfolio:</strong> {investor.portfolio.join(", ")}</p>
    </Card>
  );
}

export default InvestorProfile;
```

#### `src/pages/profile/EntrepreneurProfile.jsx`
```jsx
import { useEffect, useState } from "react";
import { useParams } from "react-router-dom";
import api from "../../services/api";
import Card from "../../components/common/Card";

function EntrepreneurProfile() {
  const { id } = useParams();
  const [entrepreneur, setEntrepreneur] = useState(null);

  useEffect(() => {
    api.get(`/entrepreneurs/${id}`).then((response) => {
      setEntrepreneur(response.data);
    });
  }, [id]);

  if (!entrepreneur) return <div>Loading...</div>;

  return (
    <Card>
      <h2 className="text-2xl mb-4">{entrepreneur.name}</h2>
      <p><strong>Startup:</strong> {entrepreneur.startup}</p>
      <p><strong>Bio:</strong> {entrepreneur.bio}</p>
      <p><strong>Pitch:</strong> {entrepreneur.pitch}</p>
      <p><strong>Funding Need:</strong> ${entrepreneur.funding}</p>
      <p><strong>Pitch Deck:</strong> <a href={entrepreneur.pitchDeck} className="text-blue-500">Download</a></p>
    </Card>
  );
}

export default EntrepreneurProfile;
```

### Week 2 Deliverables
- Investor Dashboard with entrepreneur list and profile cards.
- Entrepreneur Dashboard with collaboration requests.
- Detailed profiles for both roles.
- Mock API integration with JSON Server.
- Responsive design with Tailwind CSS.
- Push to GitHub: `git add . && git commit -m "Week 2 complete" && git push`.

---
