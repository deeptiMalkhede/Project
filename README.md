// TravelPackageDetails.tsx
const TravelPackageDetails: React.FC = () => {
    const { id } = useParams<{ id: string }>();
    const [packageDetails, setPackageDetails] = useState<TravelPackage | null>(null);
    const [reviews, setReviews] = useState<any[]>([]);
    const [showReviewForm, setShowReviewForm] = useState(false);

    useEffect(() => {
        const fetchData = async () => {
            const pkgResponse = await axios.get(`http://localhost:8080/api/packages/${id}`);
            const reviewsResponse = await axios.get(`http://localhost:8080/api/reviews/package/${id}`);
            setPackageDetails(pkgResponse.data);
            setReviews(reviewsResponse.data);
        };
        fetchData();
    }, [id]);

    const handleNewReview = () => {
        setShowReviewForm(false);
        // Refresh reviews
        axios.get(`http://localhost:8080/api/reviews/package/${id}`)
            .then(res => setReviews(res.data));
    };

    if (!packageDetails) return <div>Loading...</div>;

    return (
        <div className="container mt-4">
            <div className="row">
                <div className="col-md-8">
                    <h1>{packageDetails.name}</h1>
                    <div className="card mb-4">
                        <div className="card-body">
                            <h5 className="card-title">Package Details</h5>
                            <p>Destination: {packageDetails.destinationDTO.name}</p>
                            <p>Hotel: {packageDetails.hotelDTO.name}</p>
                            <p>Flight: {packageDetails.flightDTO.airline}</p>
                            <p>Price: ${packageDetails.price}</p>
                        </div>
                    </div>

                    <h3>Reviews</h3>
                    {reviews.map(review => (
                        <div key={review.id} className="card mb-2">
                            <div className="card-body">
                                <div className="d-flex justify-content-between">
                                    <h5>{review.user.username}</h5>
                                    <div>{'★'.repeat(review.rating)}</div>
                                </div>
                                <p>{review.comment}</p>
                                <small className="text-muted">
                                    {new Date(review.createdAt).toLocaleDateString()}
                                </small>
                            </div>
                        </div>
                    ))}
                </div>

                <div className="col-md-4">
                    <div className="sticky-top" style={{ top: '1rem' }}>
                        <div className="card mb-4">
                            <div className="card-body">
                                <h5 className="card-title">Add Review</h5>
                                <button 
                                    onClick={() => setShowReviewForm(!showReviewForm)}
                                    className="btn btn-primary mb-3"
                                >
                                    {showReviewForm ? 'Cancel' : 'Write a Review'}
                                </button>
                                
                                {showReviewForm && (
                                    <ReviewForm 
                                        entityType="PACKAGE" 
                                        entityId={packageDetails.id}
                                        onReviewSubmitted={handleNewReview}
                                    />
                                )}
                            </div>
                        </div>

                        <ReviewSection 
                            entityType="FLIGHT" 
                            entityId={packageDetails.flightDTO.id} 
                            title="Flight Reviews"
                        />
                        <ReviewSection 
                            entityType="HOTEL" 
                            entityId={packageDetails.hotelDTO.id} 
                            title="Hotel Reviews"
                        />
                        <ReviewSection 
                            entityType="DESTINATION" 
                            entityId={packageDetails.destinationDTO.id} 
                            title="Destination Reviews"
                        />
                    </div>
                </div>
            </div>
        </div>
    );
};

// Create ReviewSection component similarly for other entities
